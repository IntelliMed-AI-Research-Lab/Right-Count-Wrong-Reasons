# Right Count, Wrong Reasons

### Error Cancellation and the Real Drivers of Failure in Camera-Based Non-Invasive Video Vehicle Counting

**Research Code | Computer Vision | YOLOv8 | Vehicle Counting | Traffic Sensing**

This repository provides the experimental implementation accompanying our research on the reliability of camera-based, non-invasive vehicle counting.

Rather than proposing a new object detector, this work investigates whether accurate aggregate vehicle counts genuinely indicate reliable detections, or whether missed vehicles and false positives can cancel each other and conceal substantial errors.

The experiments examine how **vehicle density, object size, occlusion, confidence thresholds, domain adaptation, and spatial localization** influence vehicle-counting reliability and downstream traffic measurements.

---

## 1. Research Overview

Camera-based traffic monitoring offers a cost-effective and non-invasive alternative to traditional road-embedded sensors. Modern object detectors, including YOLOv8, can identify vehicles in roadside and aerial imagery and support traffic monitoring applications.

However, a detector may produce an apparently accurate vehicle count while simultaneously missing real vehicles and generating false detections.

This project investigates three central research questions:

1. Does vehicle density independently reduce detection recall after controlling for vehicle size and occlusion?
2. Can false negatives and false positives cancel each other, producing misleadingly accurate aggregate vehicle counts?
3. How sensitive are video-derived traffic density, flow, and speed estimates to detector confidence thresholds and input resolution?

The experiments combine object-level detection evaluation, statistical analysis, spatial counting diagnostics, and video-based traffic measurements.

## 2. Key Contributions

### 2.1 Confound-Aware Density Analysis

Investigates whether detection recall decreases in dense traffic scenes independently of vehicle size and occlusion.

The analysis includes:

- Sparse-versus-dense image comparisons
- Vehicle-size analysis
- Occlusion-conditioned recall
- Size–occlusion stratification
- Image-level bootstrap confidence intervals
- Logistic regression
- Comparison between pretrained and fine-tuned detectors

### 2.2 Error Cancellation in Vehicle Counting

Analyzes how missed vehicles (false negatives) and spurious detections (false positives) can compensate for one another.

Evaluation includes:

- Precision and recall
- False positives and false negatives
- Mean Absolute Error (MAE)
- Root Mean Square Error (RMSE)
- Normalized MAE
- Signed count bias
- Confidence-threshold sensitivity

### 2.3 Spatial Counting Reliability

Uses Grid-based Average Mean Absolute Error (GAME) to evaluate counting discrepancies at different spatial resolutions.

GAME is evaluated at four levels:

- GAME(0): Whole-image counting error
- GAME(1): 2 × 2 spatial grid
- GAME(2): 4 × 4 spatial grid
- GAME(3): 8 × 8 spatial grid

This analysis reveals localization errors that may remain hidden when only total vehicle counts are considered.

### 2.4 Traffic Measurement Sensitivity

Investigates how detector operating settings affect video-derived traffic measurements:

- Vehicle density: detections per frame
- Traffic flow: virtual-line crossings per minute
- Relative vehicle speed: vehicle lengths per second

The video experiments use YOLOv8 and ByteTrack, with a centroid-tracking fallback where applicable.

---

## 3. Datasets

The experiments use two camera-based vehicle datasets.

### 3.1 Top-View Highway Vehicle Dataset

**Source:** Top-View Vehicle Detection Image Dataset, Kaggle

Dataset characteristics:

- 536 training images
- 90 validation images
- Single vehicle class
- Roadside/top-view vehicle imagery
- Sample traffic video for tracking and traffic measurements

The highway detector is fine-tuned using YOLOv8n.

### 3.2 VisDrone2019-DET

**Source:** VisDrone2019 Object Detection Benchmark

Dataset characteristics:

- 6,471 training images
- 548 validation images
- 17,040 annotated vehicles in the analyzed validation subset
- Vehicle categories: car, van, truck, bus
- Object bounding boxes
- Occlusion annotations
- Truncation annotations
- Ignored regions

For the fine-tuned detector, the vehicle categories are merged into a single vehicle class.

The VisDrone experiments examine domain generalization, object size, occlusion, density-associated recall differences, and counting reliability.

**Dataset availability:** The notebook includes dataset download and preparation procedures. Dataset files are not redistributed through this repository.

---

## 4. Detection Models

The experiments use the Ultralytics YOLOv8 framework.

| Model | Configuration | Dataset |
|---|---|---|
| YOLOv8n | COCO-pretrained | VisDrone |
| YOLOv8s | COCO-pretrained | VisDrone |
| YOLOv8m | COCO-pretrained | VisDrone |
| YOLOv8n-FT | Fine-tuned | Highway |
| YOLOv8n-FT | Fine-tuned | VisDrone |

### Fine-Tuning Configuration

**Highway detector**

- Architecture: YOLOv8n
- Epochs: 100
- Image size: 640
- Batch size: 32
- Initial learning rate: 0.0001
- Dropout: 0.1

**VisDrone detector**

- Architecture: YOLOv8n
- Epochs: 35
- Image size: 960
- Batch size: 16
- Training initialization: COCO-pretrained YOLOv8n

The project compares general-purpose COCO-pretrained models against domain-specific fine-tuned detectors.

---

## 5. Methodology

The experimental pipeline consists of the following stages.

**Stage 1 — Data Preparation**

Load highway and VisDrone imagery, prepare annotations, and organize training and validation data.

**Stage 2 — Vehicle Detection**

Apply pretrained and fine-tuned YOLOv8 detectors under specified confidence thresholds and input resolutions.

**Stage 3 — Detection Matching**

Match predicted bounding boxes to ground-truth annotations using greedy one-to-one matching and an Intersection over Union threshold of 0.5.

**Stage 4 — Detection Reliability Analysis**

Calculate recall, precision, false negatives, false positives, object-size effects, occlusion effects, and image-density relationships.

**Stage 5 — Confound-Controlled Density Evaluation**

Divide vehicles into 12 size–occlusion strata and estimate standardized recall differences between sparse and dense images.

**Stage 6 — Counting Error Analysis**

Evaluate count bias, MAE, normalized MAE, RMSE, and GAME across detector confidence thresholds.

**Stage 7 — Mechanism Analysis**

Explore the effects of input resolution and non-maximum-suppression thresholds on density-associated recall differences.

**Stage 8 — Video Traffic Analysis**

Track vehicles, generate a virtual counting line, and estimate traffic density, flow, and relative speed under multiple detector operating settings.

---

## 6. Main Experimental Findings

The following findings are reported in the accompanying manuscript.

### Finding 1: Density Alone Does Not Explain Detection Failure

COCO-pretrained YOLOv8 detectors show unadjusted sparse-to-dense recall reductions of approximately 17.5–27.0 percentage points on VisDrone.

However, after controlling for vehicle size and annotated occlusion, the fine-tuned detector shows a standardized recall difference of approximately −1.3 percentage points, with a 95% confidence interval spanning zero.

Under the study's predefined decision rule, this provides no detectable independent density-associated recall loss for the fine-tuned detector.

### Finding 2: Accurate Total Counts Can Hide Detection Errors

For the fine-tuned VisDrone detector at confidence threshold 0.25:

| Metric | Result |
|---|---:|
| Net count bias | −0.4% |
| Mean Absolute Error | 4.49 |
| GAME(0) | 4.49 |
| GAME(3) | 8.39 |
| GAME(3)/GAME(0) | 1.87 |

Despite the nearly unbiased aggregate count, substantial object-level and spatial errors remain.

This demonstrates that a correct total count is not sufficient evidence of reliable detection.

### Finding 3: Confidence Thresholds Affect Counting Reliability

Count bias changes with detector confidence thresholds and differs between the highway and VisDrone datasets.

A confidence threshold that minimizes bias in one dataset does not necessarily minimize bias in another.

### Finding 4: Traffic Measurements Have Different Sensitivities

Across twelve detector operating configurations on the analyzed highway video:

| Traffic measurement | Observed range relative to reference |
|---|---:|
| Density | 57% |
| Relative speed | 17% |
| Flow | 10% |

These results characterize sensitivity to detector settings, rather than absolute measurement accuracy, because independent video-level ground truth is unavailable.

---

## 7. Repository Contents

The main implementation is provided as a Jupyter Notebook.

```text
Right-Count-Wrong-Reasons/
│
├── Traffic_Density_Project_v4_Final.ipynb
│
├── README.md
│
└── LICENSE                     # Optional
```

The notebook contains ten main parts:

| Part | Description |
|---|---|
| 1 | Setup, dataset preparation, and model training |
| 2 | Qualitative detection and tracking visualizations |
| 3 | Highway density and detection reliability analysis |
| 4 | VisDrone evaluation using COCO-pretrained detectors |
| 5 | Fine-tuned VisDrone detector and standardized density test |
| 6 | Counting metrics and exploratory mechanism analysis |
| 7 | Traffic density, flow, speed, and GAME evaluation |
| 8 | Paper figures and LaTeX table generation |
| 9 | Experimental summary and result export |
| 10 | Research notes and future directions |

Generated outputs can include plots, CSV tables, detector checkpoints, and manuscript-ready figures.

---

## 8. Installation and Requirements

### Recommended Environment

- Python 3.10 or later
- Jupyter Notebook or Google Colab
- PyTorch
- Ultralytics YOLOv8
- OpenCV
- NumPy
- Pandas
- Matplotlib
- SciPy
- Scikit-learn
- Statsmodels
- Pillow
- KaggleHub

A CUDA-compatible GPU is recommended for detector training and large-scale inference.

### Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/Right-Count-Wrong-Reasons.git
cd Right-Count-Wrong-Reasons
```

Replace `YOUR_USERNAME` with your GitHub username.

### Install Dependencies

```bash
pip install ultralytics kagglehub numpy pandas matplotlib opencv-python scipy scikit-learn statsmodels pillow jupyter
```

Install a PyTorch build appropriate for your system and CUDA environment.

### Launch the Notebook

```bash
jupyter notebook Traffic_Density_Project_v4_Final.ipynb
```

Alternatively, upload the notebook to Google Colab.

**Note:** The notebook contains environment-specific paths, dataset downloads, training stages, and computationally intensive experiments. Review the configuration cells before running the complete pipeline.

---

## 9. Reproducing the Experiments

To reproduce the experimental workflow:

1. Open the notebook and execute the environment setup.
2. Download and prepare the highway dataset.
3. Train or load the highway YOLOv8n checkpoint.
4. Run highway detection and reliability experiments.
5. Prepare the VisDrone2019-DET dataset.
6. Evaluate COCO-pretrained YOLOv8n, YOLOv8s, and YOLOv8m.
7. Fine-tune YOLOv8n on VisDrone vehicle annotations.
8. Execute size–occlusion standardized recall analysis.
9. Run confidence-threshold sweeps and GAME evaluation.
10. Execute the exploratory resolution and NMS experiments.
11. Run video tracking and traffic-measurement sensitivity analysis.
12. Export tables and figures.

Results can vary with software versions, hardware, training randomness, and model checkpoint selection.

For close reproduction, use the same datasets, configuration settings, random seeds, and detector checkpoints described in the accompanying paper.

---

## 10. Evaluation Metrics

### Recall

```text
Recall = TP / (TP + FN)
```

### Precision

```text
Precision = TP / (TP + FP)
```

### Net Count Bias

```text
Bias (%) = 100 × Sum(Predicted - Ground Truth)
                  / Sum(Ground Truth)
```

### Mean Absolute Error

```text
MAE = Mean(|Predicted Count - True Count|)
```

### Normalized MAE

```text
nMAE = MAE / Mean(True Count)
```

### Grid-based Counting Error

```text
GAME(L) = Mean over images [
    Sum over grid cells |Predicted Count - True Count|
]
```

The number of spatial cells at level L is 4^L.

### Standardized Recall

```text
Standardized Recall = Sum(w_k × Recall_k)
```

Here, each stratum represents a particular combination of vehicle size and annotated occlusion.

---

## 11. Limitations

Several limitations should be considered when interpreting the experiments:

- Fine-tuned checkpoints are selected using validation splits also employed in analysis, potentially producing optimistic absolute detection metrics.
- The video sensitivity analysis uses a short highway clip.
- Independent ground truth is unavailable for the reported video-derived density, flow, and speed measurements.
- Speed is expressed in vehicle lengths per second rather than calibrated physical units.
- Some mechanism analyses are exploratory rather than confirmatory.
- Findings should not automatically be generalized to all detectors, camera viewpoints, datasets, or traffic environments.

---

## 12. Research Significance

The central conclusion is that **aggregate counting accuracy alone is not a sufficient measure of detector reliability**.

Missed vehicles and false positives may cancel, creating misleadingly accurate totals.

Reliable camera-based traffic sensing therefore benefits from evaluating:

- Object-level detection errors
- Vehicle size and occlusion
- Confidence-threshold effects
- Spatial counting discrepancies
- Dataset and domain differences
- Downstream traffic-measurement sensitivity

The project provides an experimental framework for studying these issues using roadside and aerial vehicle imagery.

---

## 13. Citation

If you use this repository or build upon its experimental methodology, please cite the accompanying research paper.

**Paper title:**

*Right Count, Wrong Reasons: Error Cancellation and the Real Drivers of Failure in Camera-Based Non-Invasive Video Vehicle Counting*

A formal BibTeX citation will be added when publication details are available.

---

## 14. Acknowledgments

This project uses the Ultralytics YOLOv8 framework, the VisDrone2019-DET benchmark, the Top-View Vehicle Detection Image Dataset, and ByteTrack for vehicle tracking.

We acknowledge the respective developers, dataset creators, and research communities.

---

## 15. Research Status

This repository contains the experimental notebook associated with the accompanying research manuscript.

The implementation focuses on evaluating the reliability of detector-derived vehicle counts and traffic measurements rather than introducing a new detection architecture.

**Keywords:** Computer Vision, YOLOv8, Vehicle Counting, Traffic Monitoring, Non-Invasive Traffic Sensing, VisDrone, Object Detection, Error Cancellation, Occlusion, Confidence Threshold, GAME, ByteTrack, Traffic Density, Traffic Flow, Speed Estimation.
