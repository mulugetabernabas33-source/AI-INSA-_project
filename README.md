# Vehicle & License-Plate Detection

A two-stage YOLOv8 cascade pipeline for detecting vehicles and localizing their license plates in traffic-camera imagery.

![Status](https://img.shields.io/badge/status-complete-2ea44f)
![Framework](https://img.shields.io/badge/framework-Ultralytics%20YOLOv8-0f1f3d)
![Hardware](https://img.shields.io/badge/trained%20on-NVIDIA%20T4%20GPU-76b900)
![Group](https://img.shields.io/badge/group-12-C9A227)

## Results

| Vehicle mAP@50 | Plate mAP@50 | Vehicle Speed | GPU Speedup |
| -------------: | -----------: | ------------: | ----------: |
|      **96.9%** |    **95.7%** |   **104 FPS** |     **19×** |

## Overview

This project uses a **two-stage YOLOv8 cascade**:

```text
Full Frame
    ↓
Vehicle Detector
    ↓
Crop Each Vehicle
    ↓
License Plate Detector
    ↓
Plate Localized
```

### Why a Cascade?

License plates are very small objects in traffic-camera images. Detecting them directly from the full image is difficult.

The first model detects vehicles and crops them. The second model then searches for license plates inside those vehicle crops, making the plates much easier to detect.

## Detection Accuracy

| Metric        | Vehicle Model | Plate Model |
| ------------- | ------------: | ----------: |
| mAP@50        |         0.969 |       0.957 |
| mAP@50-95     |         0.771 |       0.625 |
| Precision     |         0.924 |       0.978 |
| Recall        |         0.921 |       0.931 |
| Training Time |      16.6 min |       7 min |

## Inference Speed

| Model   | Device        |   Latency |       FPS |
| ------- | ------------- | --------: | --------: |
| Vehicle | NVIDIA T4 GPU |   9.63 ms | **103.8** |
| Vehicle | CPU           | 185.51 ms |       5.4 |
| Plate   | NVIDIA T4 GPU |   7.82 ms | **128.0** |
| Plate   | CPU           |  59.06 ms |      16.9 |

GPU inference was approximately **19× faster for vehicle detection** and **7.5× faster for plate detection**.

## Dataset

| Stage   | Dataset                      | Train | Validation |
| ------- | ---------------------------- | ----: | ---------: |
| Vehicle | UA-DETRAC-DATASET-10K        | 2,000 |        500 |
| Plate   | License Plate Recognition v4 | 2,000 |        500 |

* [UA-DETRAC-DATASET-10K](https://universe.roboflow.com/rjacaac1/ua-detrac-dataset-10k)
* [License Plate Recognition v4](https://universe.roboflow.com/roboflow-universe-projects/license-plate-recognition-rxg4e)

CCPD was not used because it is focused on Chinese license plates.

The vehicle dataset classes (`car`, `bus`, `van`, `others`) were combined into a single `vehicle` class.

### YOLO Format

```text
class x_center y_center width height
```

All coordinates are normalized between `0` and `1`.

### COCO to YOLO Conversion

```text
x_center = (x_min + width / 2) / img_width
y_center = (y_min + height / 2) / img_height
width    = width / img_width
height   = height / img_height
```

## Architecture & Training

* **Model:** YOLOv8n
* **Framework:** [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)
* **Hardware:** NVIDIA Tesla T4
* **Platform:** Google Colab
* **Pretrained weights:** COCO

| Setting    | Vehicle | Plate |
| ---------- | ------: | ----: |
| Image Size |     640 |   416 |
| Batch Size |      16 |    32 |
| Epochs     |      30 |    30 |
| Patience   |       8 |     8 |
| Optimizer  |   AdamW | AdamW |

YOLOv8n was selected because it provides a good balance between speed and accuracy.

## Pipeline Examples

| Scene                         | Vehicles Found | Plates Found |
| ----------------------------- | -------------: | -----------: |
| Wide highway, distant traffic |             19 |            1 |
| Closer urban intersection     |             13 |            5 |

The lower number of detected plates in distant scenes is mainly caused by limited pixel resolution. Even after vehicle cropping, extremely small or distant plates may not contain enough visual information for reliable detection.

## Repository Structure

```text
.
├── vehicle_plate_detection.ipynb
├── Vehicle_Plate_Detection_Report.pdf
└── README.md
```

## How to Run

1. Open `vehicle_plate_detection.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Select **Runtime → Change runtime type → T4 GPU**.
3. Run all cells.
4. The notebook installs dependencies, downloads the datasets, trains both models, runs the cascade, and benchmarks CPU/GPU performance.
5. Trained weights are backed up to:

```text
/plate_project_backup/
```

## Concepts Demonstrated

* YOLO annotation format
* COCO to YOLO conversion
* Object detection
* Two-stage detection pipelines
* Small-object detection
* mAP@50 and mAP@50-95
* Precision and Recall
* CPU vs GPU inference
* Speed-accuracy tradeoffs
* YOLOv8 model training

---

**Built with Ultralytics YOLOv8 and Google Colab.**
