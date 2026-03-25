# drone-imagery-segmentation-tool
A lightweight Python tool for drone imagery segmentation using weak supervision, handcrafted features, and Random Forest classification.
# Drone Imagery Segmentation Tool

A lightweight and explainable Python pipeline for analyzing drone imagery using weak supervision.

This project processes aerial images by splitting them into tiles, extracting handcrafted visual features, generating pseudo-labels using simple heuristics, and training a `RandomForestClassifier` to classify land-cover regions into the following categories:
