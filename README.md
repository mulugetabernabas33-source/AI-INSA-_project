


Vehicle & License-Plate Detection
A two-stage YOLOv8 cascade pipeline that detects vehicles in traffic-camera imagery, then localizes license plates within each detected vehicle.

Group 12 · AI Engineering Coursework · Trained on Google Colab (Tesla T4 GPU)

Table of Contents
Overview
Results
Data
Architecture & Training
Cascade Pipeline in Action
Repository Structure
Reproducing This
Concepts Demonstrated
Overview
Instead of training a single detector to find both vehicles and plates at once, this project uses a coarse-to-fine cascade:

Full frame -> [Stage 1: Vehicle Detector] -> crop each vehicle
            -> [Stage 2: Plate Detector]   -> locate plate within crop
Why a cascade? A license plate occupies well under 1% of a full traffic-camera frame. Detecting it directly in the full image is a classic small-object detection problem. By first cropping to each vehicle, the plate becomes a much larger, easier target, turning one hard problem into two easier ones.

Results
Detection Accuracy
Metric	Vehicle Model	Plate Model
mAP@50	0.969	0.957
mAP@50-95	0.771	0.625
Precision	0.924	0.978
Recall	0.921	0.931
Training time	~16.6 min	~7 min (early-stopped)
Inference Speed — CPU vs GPU
Model	Device	Latency	FPS	Real-time (>15 FPS)
Vehicle	GPU (T4)	9.63 ms	103.8	Yes
Vehicle	CPU	185.51 ms	5.4	No
Plate	GPU (T4)	7.82 ms	128.0	Yes
Plate	CPU	59.06 ms	16.9	Yes (marginal)
GPU inference is roughly 19x faster for vehicle detection and 7.5x faster for plate detection, confirming GPU acceleration is effectively required for real-time deployment.

Data
Stage	Dataset	Source	Used
Vehicle	UA-DETRAC-DATASET-10K	Roboflow Universe	2,000 train / 500 val
Plate	License Plate Recognition v4	Roboflow Universe	2,000 train / 500 val
CCPD was deliberately avoided, as it is Chinese-plate specific and not representative of general license plates.

All labels use the YOLO format (class x_center y_center width height, normalized 0-1). The raw UA-DETRAC export contains four vehicle sub-classes (car/bus/van/others); these were remapped to a single vehicle class since the task is detection, not classification.

Architecture & Training
Model: YOLOv8n (nano), fine-tuned from COCO-pretrained weights
Framework: Ultralytics
Hardware: Google Colab, NVIDIA Tesla T4 GPU
Setting	Vehicle Stage	Plate Stage
Image size	640 px	416 px
Batch size	16	32
Epochs (max / patience)	30 / 8	30 / 8
Optimizer	AdamW (auto)	AdamW (auto)
The nano variant was chosen deliberately as a speed-accuracy tradeoff. It trains and runs fast while still achieving over 95% mAP@50 on this constrained, single-class task.

Cascade Pipeline in Action
The pipeline was tested across multiple validation images:

Scene	Vehicles Found	Plates Found	Explanation
Wide highway, distant traffic	19	1	Most vehicles too small or far for plate pixels to resolve
Closer urban intersection	13	5	Plates reliably detected near the camera
This is not a failure of the pipeline. It illustrates the small-object detection tradeoff directly: the cascade correctly narrows the search area per vehicle, but final plate visibility remains bounded by pixel resolution at capture distance.

Repository Structure
.
├── vehicle_plate_detection.ipynb        Full notebook: data prep, training, cascade, benchmark
├── Vehicle_Plate_Detection_Report.pdf   Full written report
└── README.md
Reproducing This
Open the notebook in Google Colab.
Set the runtime: Runtime -> Change runtime type -> T4 GPU.
Run all cells top to bottom. This installs dependencies, pulls both datasets from Roboflow, trains both stages, and benchmarks CPU vs GPU inference.
Trained weights are automatically backed up to Google Drive under /plate_project_backup/.
Concepts Demonstrated
COCO to YOLO annotation formats — conversion math and practical application
mAP interpretation — IoU, Precision/Recall, mAP@50 vs mAP@50-95
Small-object detection — the core reason the cascade architecture exists
Speed-accuracy tradeoffs — model size, dataset subsetting, and the measured CPU/GPU benchmark
Built with Ultralytics YOLOv8. Trained on Google Colab.


