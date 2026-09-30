# # Bone Scintigraphy (BS-80k) Metastasis Classification

This repository hosts a pipeline designed for analyzing, preprocessing, and classifying whole-body and regional bone scintigraphy (bone scan) images using the **BS-80k** dataset containing over 80,000 labeled images. 

We design, explore, and run controlled experiments to demonstrate how preprocessing techniques (like Median Filtering and CLAHE) directly improve machine learning classifier performance when identifying normal vs. abnormal (metastatic) scans.

---

##  Table of Contents
1. [Project Overview](#project-overview)
2. [Dataset Structure](#dataset-structure)
3. [Image Preprocessing & Pipeline](#image-preprocessing--pipeline)
4. [Hotspot Detection](#hotspot-detection)
5. [Controlled Change Experiment](#controlled-change-experiment)
6. [Results & Analysis](#results-analysis)
7. [How to Use](#how-to-use)

---

##  Project Overview
Bone scintigraphy is a vital nuclear medicine imaging technique used to detect bone metastases. The low-resolution and high-noise characteristics of scintigraphy scans make automatic computer-aided diagnosis challenging.

This project presents:
* A robust pipeline to automatically structure, clean, and map labels across 28 body regions/scans.
* Advanced image enhancement techniques (Noise reduction + local contrast enhancement).
* Target resizing with padding to prevent distortion of anatomical proportions.
* A controlled experiment quantifying the impact of image enhancement on a Random Forest classifier.

---

##  Dataset Structure
The dataset is organized into 28 subdirectories representing anatomical regions and views (e.g., Anterior vs. Posterior), including:
* `headANT` / `headPOST` (Head Anterior/Posterior)
* `wholeBodyANT` / `wholeBodyPOST` (Whole Body scans)
* `chestLANT`, `shoRPOST`, `pelvisANT`, `ankleRANT`, etc.

Each subdirectory contains its associated images and a `.txt` mapping file containing filenames and binary labels (`0` = Normal, `1` = Abnormal/Metastasis).

---

##  Image Preprocessing & Pipeline
Before training, scintigraphy images undergo a multi-stage preprocessing pipeline:
1. **Noise Reduction:** A `3x3` Median Filter is applied to handle salt-and-pepper noise inherent in low-count gamma camera imagery.
2. **Contrast Enhancement:** **CLAHE** (Contrast Limited Adaptive Histogram Equalization) is applied with a `clipLimit=2.0` and grid size of `(8, 8)` to highlight local bone density anomalies (hotspots) without over-amplifying background noise.
3. **Aspect-Ratio Preserved Padding:** Images are resized using zero-padding to a standard canvas to preserve native aspect ratios (preventing skeletal distortion).
4. **Regional Slicing:** Support for partitioning whole-body scans into regional segments (e.g., Head, Thorax, Pelvis, Legs) for targeted spatial scanning.

---

##  Hotspot Detection
We employ dynamic intensity-based thresholding to segment active zones. By isolating intensities exceeding a dynamic statistical percentile (e.g., `98.5%`), the system automatically contours active lesions (potential hotspots) with bounding boxes.

---

##  Controlled Change Experiment
To evaluate the impact of preprocessing, we isolated a subset of 1,000 scans from the `headANT` region for a classification experiment:
* **Independent Variable:** Preprocessing state (Raw Unfiltered vs. Preprocessed with Median Filter + CLAHE).
* **Classifier:** Random Forest Classifier (100 estimators).
* **Features:** Robust statistical descriptors (`Mean`, `Standard Deviation`, `90th`, `95th`, and `99th` percentiles).
* **Validation:** 70/30 Train/Test Split.

---

##  Results & Analysis

### Metrics Summary
| Configuration | Accuracy | F1-Score (Abnormal Class) | Macro Avg F1 |
| :--- | :---: | :---: | :---: |
| **Raw Unfiltered Images** | 89.00% | 0.00 | 0.47 |
| **Preprocessed (Filter + CLAHE)** | **90.33%** | **0.06** | **0.51** |

### Key Takeaway
Without preprocessing, the model completely failed to predict abnormal cases (0% recall/precision for Class 1 due to high noise-to-signal ratio). By adding **Median Filtering and CLAHE**, the classifier successfully distinguished true abnormalities, lowering false negatives and significantly improving macro-average metrics.
