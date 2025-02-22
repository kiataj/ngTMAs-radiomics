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
Key parameters configuration, activating/deactivating filter classes or feature classes can be done in the extraction [Extraction](https://github.com/kiataj/ngTMAs-radiomics/blob/main/Extraction.ipynb) notebook.
For more detailed documentation check out the [Extraction](https://github.com/kiataj/ngTMAs-radiomics/blob/main/Extraction.ipynb) notebook.

## Feature Elimination

The TMAs are now embedded in a high dimensional space, with many of its dimensions being either rudimentary or non-reproducible, i. e. the measurement and calculation of that feature cannot be repeated due to the varying imaging parameters, noise, or varying fixation or embedding protocol. 

### Non-reproducible features
We use [intraclass_corr](https://pingouin-stats.org/build/html/generated/pingouin.intraclass_corr.html) from pinguin library between two distinct measurements, one in the  vertical position and another in a horizontal position. The scanning conditions' characteristics are fully discussed here (https://ieeexplore.ieee.org/abstract/document/10542137). Then the extracted features from these two scan modes are used for the calculation of intraclass correlation (ICC). A threshold for selecting robust features can be selected:

| ICC Value     | Reliability Level  |
|--------------|--------------------|
| < 0.50       | Poor reliability   |
| 0.50 – 0.75  | Moderate reliability |
| 0.75 – 0.90  | Good reliability   |
| > 0.90       | Excellent reliability |

[ICC](https://github.com/kiataj/ngTMAs-radiomics/blob/main/ICC.ipynb) notebook can be used for calculating ICC value for the features. First features with zero variance are removed, then a sign preserving Log-transform is performed to rescale the features and bring the features closer to a normal distribution. Then the batch effect in the extraction is corrected so the correlations are not affected by batch effect, and finaly ICC values are calculated. 

### Redundant features
Pairs of the features are picked up and pearson correlation corefficient is calculated between each pair, if the correlation coefficient is larger than a threshold, one of the two features is randomly discarded. This process can be done with [redundancy reduction](https://github.com/kiataj/ngTMAs-radiomics/blob/main/Redundancy%20reduction.ipynb) notebook. First selected features from the previous step (ICC) are retained for subsequent processing, a sign-preserving log transformation is performed, and the batch effect is corrected. Then pairwise Pearson r is calculated and redundant features are removed. **Note that, only the redundant features should be used from this notebook. Any other data processing in ICC and redundancy reduction notebook such as log transformation, and batch correction should not be transferred to the classification as they are a source of data leakage. The sole purpose of ICC and redundancy notebooks are to discard the bad features and retain the good ones.**

## Feature processing
The emebeddings need to be processed before being passed for inference. 

### Skewness (Log Transform)
Some features might have high Skewness, which is contrary to many priors we use in statistical analysis. To correct this we can use log transform on those features. 

### Normalization (z-score)
Some features might have higher values, which results in higher variance, and therefore their proportion of affecting certain statistical calculations is stronger. This can be taken care of by centering features mean value at zero with the standard deviation of one.

### Batch Correction (ComBat)
Batch effect can arise from variations in the measurements, it can be because of imaging settings, or embedding support (paraffin) variations. It can be corrected using ComBat which models each batch by a linear model, that fits the observed values of each feature in each batch to a constant, a mean shift, a variance shift, and random noise. It learns the mean shift and variance shift of each batch then removes them. <br>
Reference: Behdenna A, Haziza J, Azencot CA and Nordor A. (2020) pyComBat, a Python tool for batch effects correction in high-throughput molecular data using empirical Bayes methods. bioRxiv doi: 10.1101/2020.03.17.995431 <br>

### Visualization (UMAP)
[UMAP](https://umap-learn.readthedocs.io/en/latest/parameters.html)
Reference: McInnes, Leland, John Healy, and James Melville. "Umap: Uniform manifold approximation and projection for dimension reduction." arXiv preprint arXiv:1802.03426 (2018).
## Classification

### Classifier
### Evaluation
