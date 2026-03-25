# drone-imagery-segmentation-tool
A lightweight Python tool for drone imagery segmentation using weak supervision, handcrafted features, and Random Forest classification.
# Drone Imagery Segmentation Tool

A lightweight and explainable Python pipeline for analyzing drone imagery using weak supervision.

This project processes aerial images by splitting them into tiles, extracting handcrafted visual features, generating pseudo-labels using simple heuristics, and training a `RandomForestClassifier` to classify land-cover regions into the following categories:

- Vegetation
- Water
- Soil
- Roads
- Built Structures
The solution is intentionally designed to be **simple, interpretable, and practical**, making it suitable for rapid analysis when labeled training data is unavailable.

---
## Overview

In many drone-imagery workflows, labeled datasets are expensive to create and difficult to maintain across varying geographies, lighting conditions, and sensors.
To address this, this project uses a **weakly supervised classical machine learning pipeline** that:
- avoids dependency on annotated datasets,
- remains computationally lightweight,
- produces interpretable outputs,
- and is easy to extend or replace with stronger models later.

This makes it a strong baseline for quick experimentation, land-cover prototyping, or assignment-scale remote sensing tasks.

---
## Methodology

The pipeline follows a five-stage workflow:
### 1. Image Tiling
The input image is divided into fixed-size square patches (default: `64x64` pixels).

This keeps processing manageable and allows the pipeline to operate consistently on both standard images and large aerial maps.
### 2. Feature Extraction
Each tile is converted into a compact feature vector using lightweight handcrafted descriptors:
- **Color statistics**
  - Mean and standard deviation of RGB channels
  - Mean and standard deviation of HSV channels
- **Texture**
  - Shannon entropy to estimate local visual complexity
- **Structure**
  - Edge density using Canny edge detection
 
    These features were chosen to balance interpretability, speed, and reasonable discrimination across common land-cover types.

### 3. Pseudo-Label Generation
Because no labeled dataset is assumed, the pipeline assigns initial labels using heuristic rules derived from tile characteristics.

Examples:
- high green dominance → likely vegetation
- low brightness / blue-heavy regions → potential water
- high edge density + neutral tones → possible built structures or roads

These pseudo-labels are intentionally approximate and act as weak supervision rather than ground truth.
### 4. Model Training
A `RandomForestClassifier` is trained on the pseudo-labeled feature set.

This step allows the system to:
- smooth out hard heuristic boundaries,
- learn more flexible decision rules,
- and produce more consistent tile-level predictions than fixed rules alone.
- ### 5. Prediction and Reconstruction
The trained classifier predicts a class and confidence score for every tile.

The tile predictions are then reconstructed into:
- a full classified land-cover map,
- a visualization overlay on the original image,
- and a confidence heatmap.

---
## Key Design Decisions

This project intentionally uses a **classical ML + heuristics** approach rather than deep learning for the following reasons:

- **No dependency on labeled data**
- **Fast iteration and low setup overhead**
- **Transparent, explainable decision flow**
- **Lightweight execution on standard machines**
- **Appropriate complexity for assignment-scale evaluation**
- While a CNN-based solution may improve accuracy with labeled data, this implementation prioritizes clarity, reliability, and practical constraints.

---

## Supported Input Formats

The pipeline supports the following image formats:
- `PNG`
- `JPG`
- `JPEG`
- `TIF`
- `TIFF`
- ### TIFF / GeoTIFF Handling
- `rasterio` is preferred for `TIF/TIFF` files, especially large or geospatial TIFFs.
- OpenCV is used for standard image formats and simple TIFF-compatible cases.
- For large GeoTIFF inputs, resizing is recommended to reduce memory usage and improve runtime.

---

## Project Structure

```text
drone_ai_assignment/
├── data/               # Input images (e.g., Drone_SAMPLE.tif)
├── outputs/            # Generated predictions and visualizations
├── src/                # Source code modules
├── main.py             # Entry point for the pipeline
├── requirements.txt    # Python dependencies
└── README.md           # Project documentation
````

---
## Installation

### Requirements

* Python **3.8+**
* `pip`

### Setup

1. Clone or download the project.
2. Open a terminal in the project root.
3. Install dependencies:
