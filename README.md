---
title: CHM_DINOv3 - Data Science Group
purpose: Notebook-based fine-tuning a CHM DINOv3 model (satellite-large, 300M parameter) based on SR-lite images (NRG bands used) and G-LiHT CHM
---

# CHM_DINOv3

Repo for Files associated with DINOv3 CHM fine-tuning

## Objectives

- Test DINOv3 foundation model for earth-observing satellite imagery. Apply model to Alaska's taiga-tundra ecotone

## Containers

### Notebook runs on DEV kernel

## Quickstart

- 2 methods:
- 1) Have a 2 directory of images and labels, with chips that have matching names and sizes in each directory. From here can directly run "3_DINOv3_fulldataset.ipynb"
  2) Have a huggingface dataset (or create one with "1_chips_to_include.ipynb" and "2_create_dataset.ipynb") then run "3_DINOv3_huggingface.ipynb"

## Dataset Generation and Training

### Notebooks 1-chips_to_include identifies the chips to use in the training (and can filter based on geolocation and statistical metrics) and creates a CSV to pass to 2-create_dataset which makes a huggingface dataset of the images and labels and calculates the mean & std of the channels

## Full Data Pipeline Command

### 0_Dinov3_Model is the entire command to process the data, create preview visualizations, train model, evaluate performance, and inference on SR-lite raster files

## Contributors

### Melanie Frost, Paul Montesano, Matt MacCander, JJ Frost
