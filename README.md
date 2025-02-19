# ngTMAs-radiomics
This repository contains essential processes for processing ngTMAs micro-CT images for radiomics and deep learning applications.


## Filters

We use [pyradiomics](https://pyradiomics.readthedocs.io/en/2.0.1/index.html) for radiomics feature extraction, where filter classes are computed and applied during extraction if enabled. However, in ngTMAs, air inclusions and embedding artifacts can introduce dominant effects in filtered images, such as gradients, compromising normalization. This, in turn, can cause discretization to remove critical information. To mitigate this, we manually compute and normalize filtered images before using them as input for feature extraction.

