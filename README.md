# PCB Defect Detection and Classification

YOLOv8-based detector for six common PCB manufacturing defects, with a Streamlit demo for image upload and boxed results.

Built during an Infosys Springboard AI internship. The work went through two approaches: patch classification first, then full-image object detection after the first method failed in the web interface.

**Dataset and trained weights :**  
https://drive.google.com/drive/folders/1jckJoBLvpNfM4YSmr34En9ao4PbfXl9F?usp=sharing

**Repository:**  
https://github.com/satya-suraj-pedada-ds/SURAJ-Infosys-Springboard-Projects

---

## Problem

Manual PCB inspection is slow and inconsistent. A useful system must do two things on one photo:

1. Find where the defect is
2. Name the defect type

Six classes:

| Class | Meaning |
|---|---|
| missing_hole | drilled hole is absent |
| mouse_bite | small bite / erosion on a track |
| open_circuit | broken copper track |
| short | extra connection between tracks |
| spur | thin extra copper protrusion |
| spurious_copper | leftover copper island |

---

## Why the approach changed

### Phase 1 — Milestone 1 (patch classification)

Internship guidance started with **EfficientNet-style classification** on small crops.

Pipeline in the Milestone 1 notebooks:

- Load a defective board from `images/`
- Load the matching clean template from `PCB_USED/` (`01.JPG` … `12.JPG`)
- Resize, grayscale, blur, adaptive threshold
- Subtract template from test image
- Find contours and save **64×64 grayscale patches**

Those patches are in Drive under `PCB_USED/<class>/` as files like `missing_hole_defect_0000.png`.

Classification accuracy on those crops looked high (around 0.90). That number only answers “what is this already-cropped patch?”

It did **not** work as expected in the web UI, because the UI receives a **full PCB image**, not a ready 64×64 crop. A classifier cannot draw reliable boxes on the whole board.

### Phase 2 — Milestone 2 (object detection)

After that failure, the recommended change was **YOLOv8**.

YOLO is trained on full images plus official XML boxes. It outputs bounding boxes and class names in one step. That matches what the Streamlit app needs.

Phase 1 patches were **not** used to train YOLO. They belong to the first attempt only.

---

## Final system
PCB image
→ YOLOv8n (conf=0.25, IoU=0.45)
→ boxes + class + confidence
→ Streamlit UI (upload, table, download)

Training setup used in the notebook:

- Model: YOLOv8n
- Image size: 640
- Batch: 16
- Epochs: 80 (patience=15)
- Device: CPU
- Runtime: about 12 hours
- Split: 70% train / 15% val / 15% test on 693 labeled images

Label conversion: Pascal VOC XML → YOLO `class cx cy w h` (normalized).

---

## Results (from the training / test notebooks)

Primary metric is **mAP@0.5** (detection), not classification accuracy.

### Test set

| Metric | Value |
|---|---|
| mAP@0.5 | 0.839 |
| mAP@0.5:0.95 | 0.442 |
| Precision (mean) | 0.881 |
| Recall (mean) | 0.748 |
| F1 (mean) | 0.804 |
| Images / instances | 104 / 446 |

### Per-class test (mAP@0.5)

| Class | Precision | Recall | mAP@0.5 |
|---|---|---|---|
| missing_hole | 1.000 | 0.983 | 0.991 |
| mouse_bite | 0.861 | 0.607 | 0.740 |
| open_circuit | 0.754 | 0.754 | 0.837 |
| short | 0.911 | 0.810 | 0.886 |
| spur | 0.870 | 0.566 | 0.739 |
| spurious_copper | 0.891 | 0.766 | 0.842 |

### Validation set (for reference)

| Metric | Value |
|---|---|
| mAP@0.5 | 0.809 |
| mAP@0.5:0.95 | 0.393 |

Missing holes are large and consistent, so the model is strongest there. Spurs and mouse bites are small track defects, so recall is lower. That is why the overall mAP sits near **0.84** instead of the easy-class score.

Sample boxed outputs are in `Results/` in this repo and in Drive `results/yolo_red_boxes/`.

---

## Repository layout
```text
SURAJ-Infosys-Springboard-Projects/
├── README.md
├── PCB-Milestone-1(Module-1 and Module-2)/
│   ├── Missing_Holes.ipynb
│   ├── Mouse_Bite.ipynb
│   ├── Open_Circuit.ipynb
│   ├── Short.ipynb
│   ├── Spur.ipynb
│   └── Spurious_Copper.ipynb
├── PCB-Milestone-2(Module-3 and Module-4)/
│   ├── Milestone-2 _ Module 3.ipynb
│   └── Milestone-2 _ Module 4.ipynb
├── PCB-Milestone-3(Module-5 and Module-6)/
│   └── Milestone-3 _ Module-5-and-Module-6.ipynb
├── PCB-Milestone-4(Module-7 and Module-8)/
│   ├── AUTOMATED PCB DEFECT DETECTION.docx
│   └── AUTOMATED PCB DEFECT DETECTION.pptx
└── Results/
    ├── Sample_Missing_Holes_Detections/
    ├── Sample_Mouse_Bite_Detections/
    ├── Sample_Open_Circuit_Detections/
    ├── Sample_Short_Detections/
    ├── Sample_Spur_Detections/
    └── Sample_Spurious_Copper_Detections/
```

