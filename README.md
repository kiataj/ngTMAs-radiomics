# ngTMAs-radiomics
This repository contains essential processes for processing (next generation tissue micro arrays) ngTMAs micro-CT images for radiomics and deep learning applications. The processes are done for tif files but should be applicable to any other format as long as they are supported in [SimpleITK](https://pypi.org/project/SimpleITK/).

## Segmentation

We need to segment the stack and encode the coordinates in the gray values of the generated mask. This is achieved through the [Segmentation](https://github.com/kiataj/ngTMAs-radiomics/blob/main/Segmentation.ipynb). It uses [Hue's cricle transform](https://docs.opencv.org/3.4/d4/d70/tutorial_hough_circle.html). Then each detected circle is assigned a coordinate by comparing against a grid on the image made by the number of rows and columns, so it is necessary to provide the exact number of rows and columns, counting is based on natural numbers. Each detected circle is then duplicated along the the third dimension, to create a stack mask.

## Image processing

Radiomics feature extraction requires a set of preprocessing steps which are covered in the [ImageProceessing](https://github.com/kiataj/ngTMAs-radiomics/blob/main/ImageProcessing.ipynb) notebook. In this section you can find a bried description of what is covered in this notebook.

### Filters

We use [pyradiomics](https://pyradiomics.readthedocs.io/en/2.0.1/index.html) for radiomics feature extraction, where filter classes are computed and applied during extraction if enabled. However, in ngTMAs, air inclusions and embedding artifacts can introduce dominant effects in filtered images, such as gradients, compromising normalization. This, in turn, can cause discretization to remove critical information. To mitigate this, we manually compute and normalize filtered images before using them as input for feature extraction.


### Histogram matching

A more objective approach for normalizing images and subsequent filtered images is to use [histogram matching](https://simpleitk.org/doxygen/latest/html/classitk_1_1simple_1_1HistogramMatchingImageFilter.html), where a reference image histogram is used as a templated for the histogram of the to be normalized images. <br>
Reference: Laszlo G. Nyul, Jayaram K. Udupa, and Xuan Zhang, "New Variants of a Method of MRI Scale Standardization", IEEE Transactions on Medical Imaging, 19(2):143-150, 2000.

### Histogram Stretching

Instead of manually stretching the histograms a cell is written that goes through all the subfolders of the given path and if it find a file with the specified name, it does perform histogram stretching by the minimum and maximum values specified by the user. 

### Discritization

Images are discritized to a few gray value intensities to avoid sparse matrices in second order feature extraction. Discritization can be used for visual inspection of the images that are actually used for feature extraction.

## Feature Extraction

Feature extraction is performed using PyRadiomics. Extraction is parallelized on CPU only across filters, meaning single-image extraction is not parallelized. <br>

A few key parameters must be set:<br>

**Normalization:** Set to False since normalization is handled during image preprocessing. <br>
**preCrop:** Set to False, since it has no effect unless filters are activated. <br>
**binWidth:** A recommended value is 32, resulting in 8 gray values. <br>

The ExtractFeatures() function requires four inputs: <br>

**extractor** – A PyRadiomics object with preconfigured settings. <br>
**Address** – Path to the folder containing subfolders of TMAs. <br>
**TMAs** – A list of TMA subfolder names to process (e.g., ['TMA1', 'TMA2']). <br>
**info** – A dictionary containing: <br>
<pre> **Block** – A list of TMA names (stored in the results CSV for reference). <br>
<pre> **Grid** – A list of grid names in the TMAs (stored in the results CSV). <br>
<pre> **filters** – A list of image filters to apply during extraction: <br>

['original', 'logarithm', 'gradient', 'squareroot', 'square', 'exponential', 
 'log-sigma-2-mm-3D', 'wavelet-HHL', 'wavelet-HLH', 'wavelet-HLL', 
 'wavelet-LHH', 'wavelet-LHL', 'wavelet-LLH', 'wavelet-LLL']
