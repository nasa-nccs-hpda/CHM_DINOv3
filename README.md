---
title: CHM_DINOv3 - Data Science Group
purpose: Notebook-based fine-tuning a CHM DINOv3 model (satellite-large, 300M parameter) based on SR-lite images (NRG bands used) and G-LiHT CHM
---

# CHM_DINOv3

Repo for files associated with DINOv3 CHM fine-tuning

## Objectives

- Test DINOv3 foundation model for earth-observing satellite imagery. Apply model to Alaska's taiga-tundra ecotone

## Containers

### Notebook runs on DEV kernel

## Quickstart

2 Data Structures:
  1) Have a 2 directory of images and labels, with chips that have matching names and sizes in each directory. From here can directly run "DINOv3_CHM.ipynb"
  2) Have a huggingface dataset and load using:
     from datasets import load_dataset, load_from_disk
     dataset = load_from_disk("/filepath")

## Dataset Generation and Training

### 

a. Datset generation

Recommend: Preprocess data by running image data through https://github.com/nasa-nccs-hpda/srlite process, then scale data to 0-1 while removing outliers and record global mean and std dev statistics. Ours were:
Mean values per channel (NIR, Red, Green): [0.34176117 0.08127703 0.10034456]
Standard deviation per channel (NIR, Red, Green): [0.13601232 0.04182002 0.03420006]
create_dataset.ipynb in "notebooks" has more information

Our dataset consisted of 499,284 pairs of 4-band (near-infrared [NIR], red, green, and blue) image and gridded vegetation height (CHM) chips in Alaska from the years 2011-2022, both stored as arrays. Alaska’s biomes fall in the boreal-tundra gradient, and each image chip generally shows one of the following categories:  tundra, sparse woody vegetation, dense woody vegetation, and mixed vegeatation. The vegetation height chips were gridded height estimates collected from Goddard LiDAR, Hyperspectral, and Thermal Imager (G-LiHT) Light Detection and Ranging (lidar) and United States Geological Survey (USGS)’s Interferometric Synthetic Aperture Radar (SAR) databases. 

<img width="524" height="362" alt="image" src="https://github.com/user-attachments/assets/b73c2edc-8bba-4bd3-a2a1-2c547d2d6b32" />

The lidar CHM chips have better fidelity but are limited to Alaska’s interior and generally dense wooded areas, where the SAR CHM chips are not as reliable, but cover a much larger geographic area and more varied vegetation than the lidar CHM chips. 
The paired optical images are originally from Maxar WorldView2 and WorldView3 top-of-atmosphere very-high-resolution (2 m) satellite images. Only observations from July and August were collected to maximize vegetation detection. 
B.	Data preprocessing
The images were standardized through an established process where first the top-of-atmosphere images were orthorectified and reprojected to the local Universal Transverse Mercator coordinate system [9]. Then, a cloud cover mask and Landsat-derived surface-reflectance reference were generated to apply a cloud mask and band-wise surface reflectance correction to the images. The output of these steps is estimated surface-reflectance (SR) earth-observing images.
The CHM and SR images were matched by time and geography, then patched into 64x64-pixel chips, where each pixel represents 2 meters. The data was first filtered for missing values, then we removed all chip pairs that did not have a single vegetation height pixel above 0 (removing 25,494 pairs for incomplete data and 45,326 pairs for no positive values). After an 80/20 train/validation split, this left 342,771 sample pairs for training and 85,693 for validation. To calculate per-band statistics and manage outliers, we calculated statistics from 1,000,000 randomly sampled points from the image chips and clamped the bands from 0 to mean + 3 standard deviations. These calculated statistics were saved as global variables for application in later steps of the model application. 
Computer vision models typically z-score normalize input images before training; we used the calculated band means and standard deviations because remote sensing bands have different distributions than web images. Finally, in early iteration NIR-red-green bands predicted as well as all four bands in the model, so we dropped the blue band to save on dataset and compute in the final versions of the model training. To facilitate early development, we made a few subsamples of the dataset for model testing, including lidar CHM- and SAR CHM-only datasets and sample sizes of 10,000, 50,000, and 100,000 pairs and the full dataset. The smaller datasets allowed us to rapidly test model structure, hyperparameters, and loss functions.


## Full Model

Link to Meta's official DINOv3 release: https://ai.meta.com/dinov3/

### Dinov3_CHM.ipynb is the entire command to z-score normalize, create preview visualizations, train model, evaluate performance, and inference on SR-images

Some of the model training structures remained stable throughout the building process. Early on we decided to run for 100 epochs with early stopping if there was no improvement in the loss function after 10 epochs. For the initial patch embeddings, we duplicated the red band’s embeddings for the NIR band. Training the 7B-parameter model caused the V100s to run out of memory, so we proceeded with using the 300M “large” model as a frozen backbone and only fine-tuned with our datasets instead of re-training the whole model. 
Among the most consequential adaptations was going from a relatively modest depth decoder with four progressive channel reduction layers, to including a more robust decoder including leaky ReLU layers, dropouts, batch norm, skip connections, and channel attention in later layers. We found that these additions helped to reduce overfitting, make the model more generalizable over the various biomes in Alaska, and better able to detect fine details in vegetation height. The training notebook can be downloaded from the public github repository: https://github.com/nasa-nccs-hpda/CHM_DINOv3 
We evaluated the learning rate performance by comparing the training and validation losses for each epoch. The final learning rate configuration started at 5e-5 and updated using a learning rate scheduler with the AdamW optimizer.
Because the study area’s vegetation height was predominantly short shrubs or barren land, the underlying data was highly skewed, and the models tended to underestimate the small number of tall trees in the study area. In addition to these architecture refinements, three more adjustments helped the model to discern taller trees while still detecting realistic canopy height variability of short shrubs:
•	an asymmetric mean-squared-error loss function that penalized underestimation by a factor of 3
•	including more data (the full dataset predicted taller trees better than early training runs of 10k – 100k image-CHM pairs)
•	setting the initial output layer to the 90th percentile of CHM datapoints (8.42m).
The final major deviation from the standard DINOv3 model was to maintain the original 64x64 size throughout, instead of resizing to the expected 224x224 size. This had multiple advantages, including greatly reducing computational memory requirements and preserving detail for the prediction. The final model training run including all the chips took approximately 11 hours for 47 epochs on the 4 GPUs. 

## Contributors

### Melanie Frost, Paul Montesano, Matt Macander, Jordan Caraballo-Vega, JJ Frost
