# ngTMAs-radiomics
This repository contains essential processes for processing ngTMAs micro-CT images for radiomics and deep learning applications. The processes are done for tif files but should be applicable to any other format as long as they are supported in [SimpleITK](https://pypi.org/project/SimpleITK/).

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
