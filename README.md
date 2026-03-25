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
4. 

```bash
pip install -r requirements.txt
```

> **Note:**
> `rasterio` is recommended for TIFF / GeoTIFF support.
> If installation is problematic on a local machine, the project can still run on standard image formats using OpenCV.

---

## Usage

Run the pipeline from the command line.

### Basic Example

```bash
python main.py --input data/Drone_SAMPLE.tif --output-dir outputs
```

### Recommended Example for Large TIFF Inputs

```bash
python main.py --input data/Drone_SAMPLE.tif --tile-size 64 --resize-width 1024 --resize-height 1024 --output-dir outputs
```

This resized mode is recommended for large drone imagery because it:

* reduces memory pressure,
* speeds up feature extraction,
* and keeps output generation predictable.

---

## Command-Line Arguments

| Argument          | Type  | Required | Default   | Description                                |
| ----------------- | ----- | -------- | --------- | ------------------------------------------ |
| `--input`         | `str` | Yes      | —         | Path to the input image file               |
| `--output-dir`    | `str` | No       | `outputs` | Directory where output files will be saved |
| `--tile-size`     | `int` | No       | `64`      | Size of square tiles in pixels             |
| `--resize-width`  | `int` | No       | `None`    | Optional resize width before processing    |
| `--resize-height` | `int` | No       | `None`    | Optional resize height before processing   |

---

## Output Artifacts

The pipeline generates the following files in the output directory:

### 1. `predictions.csv`

Tile-level structured predictions containing:

* `tile_id`
* `x`
* `y`
* `predicted_class`
* `confidence`

This file is useful for downstream analysis, validation, or integration with external tools.

### 2. `classified_map.png`

A color-coded land-cover segmentation map reconstructed from tile predictions.

### 3. `classification_overlay.png`

The classified map blended with the original image to provide visual context and improve interpretability.

### 4. `confidence_heatmap.png`

A confidence visualization showing where the model is more or less certain about its predictions.

---

## Example Execution (Validated)

The pipeline was successfully tested on a large TIFF drone image using the following command:

```bash
python main.py --input data/Drone_SAMPLE.tif --tile-size 64 --resize-width 1024 --resize-height 1024 --output-dir outputs
```

### Successful outputs generated:

* `predictions.csv`
* `classified_map.png`
* `classification_overlay.png`
* `confidence_heatmap.png`

This confirms that the full pipeline executes end-to-end successfully on a real TIFF input.

---

## Technical Assumptions

This implementation makes the following practical assumptions:

1. **Broad land-cover categories are visually separable at tile scale**
   The approach assumes that classes such as vegetation, soil, roads, and built structures have sufficiently distinct color / texture signatures in many cases.

2. **Heuristic pseudo-labels are noisy but directionally useful**
   The initial labels are not treated as ground truth. They are expected to be imperfect, but useful enough to bootstrap a simple classifier.

3. **A resized working resolution is acceptable for assignment-scale inference**
   For very large source imagery, resizing is used as a pragmatic tradeoff between speed, memory, and spatial detail.

---

## Limitations

### 1. Heuristic Sensitivity

Pseudo-label quality depends heavily on image appearance.

Performance may degrade if:

* lighting conditions are extreme,
* the image is captured at night or dusk,
* strong color grading is present,
* or terrain classes visually overlap.

### 2. Memory Footprint

The pipeline processes the full working image in memory after loading.

For very large source imagery, especially high-resolution TIFFs, this may become expensive on machines with limited RAM.

### 3. Tile Boundary Artifacts

Because predictions are tile-based, class transitions may appear blocky or overly sharp.

This is a known tradeoff of fixed-grid inference and is common in simple patch-based segmentation pipelines.

### 4. No Spatial Context Between Tiles

Each tile is processed independently.

This means the classifier does not explicitly model:

* neighborhood continuity,
* object shape,
* or larger scene context.

---

## Potential Improvements

The current implementation is intentionally lightweight, but there are several clear upgrade paths:

### 1. Overlapping Tiles / Sliding Window

Use a stride smaller than the tile size to reduce blocking artifacts and improve boundary quality.

### 2. Chunked TIFF Processing

Process very large aerial imagery in windows instead of loading the full image at once.

This would improve scalability for production-sized maps.

### 3. Richer Texture Features

Add more discriminative handcrafted descriptors such as:

* GLCM texture metrics
* Local Binary Patterns (LBP)
* gradient histogram features

### 4. Deep Feature Extraction

Replace or augment handcrafted features with embeddings from a lightweight CNN such as:

* MobileNet
* EfficientNet-lite

This would likely improve robustness if higher accuracy is required.

### 5. Supervised Training Mode

Support training on real labeled tiles or masks when annotations are available.

This would allow the same project structure to evolve from:

* heuristic baseline → weak supervision → fully supervised workflow

---

## Why This Approach Works Well for This Assignment

This solution is a good fit for an assignment setting because it demonstrates:

* practical Python engineering
* image preprocessing
* feature engineering
* weak supervision / pseudo-labeling
* classical machine learning
* structured output generation
* visualization for interpretability
* reasonable handling of large TIFF imagery

It is intentionally designed to be:

* lightweight,
* explainable,
* easy to run,
* and easy to discuss in a technical interview.

---

## Summary

This project implements a practical and interpretable baseline for drone imagery segmentation using:

* tile-based preprocessing,
* handcrafted feature extraction,
* heuristic pseudo-label generation,
* Random Forest classification,
* and visualization of model outputs.

While it is not intended to replace a fully supervised deep learning segmentation pipeline, it provides a strong, efficient starting point for land-cover analysis when labeled data is unavailable.


