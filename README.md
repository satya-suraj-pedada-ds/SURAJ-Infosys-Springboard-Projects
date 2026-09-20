# PCB Defect Detection and Classification

YOLOv8-based detector for six common PCB manufacturing defects, with a Streamlit demo for image upload and boxed results.

Built during an Infosys Springboard AI internship. The work went through two approaches: patch classification first, then full-image object detection after the first method failed in the web interface.

**Dataset and trained weights (view only):**  
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
