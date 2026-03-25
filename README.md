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
