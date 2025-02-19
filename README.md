# ngTMAs-radiomics
This repository contains essential processes for processing ngTMAs micro-CT images for radiomics and deep learning applications.


## Filters

We use [pyradiomics](https://pyradiomics.readthedocs.io/en/2.0.1/index.html) for radiomics feature extrcation, were filter classes are computed and extracted within the extraction process if enabled. However, in case of ngTMAs, air inclusion and embedding artifacts can lead to dominant effects in filtered images like gradient. This leads to ineffective normalization after filtering the image, which after discrtization removes all the information. To avoid this we must manually calculate filtered images and normalize them. Then the filtered images are fed as original image to the extraction process.

