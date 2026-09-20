
Start with Milestone 2 if you only care about the detector that the app uses.

---

## What is in the Google Drive folder

Drive folder name: `PCB_DATASET`  
Link (Anyone with the link → Viewer):  
https://drive.google.com/drive/folders/1jckJoBLvpNfM4YSmr34En9ao4PbfXl9F?usp=sharing

### Original dataset files

| Path | Role |
|---|---|
| `images/` | Raw defective PCB photos (6 class folders) |
| `Annotations/` | Official XML bounding boxes |
| `PCB_USED/01.JPG` … `12.JPG` | Clean template boards |
| `rotation/`, `rotate.py` | Official rotated copies from the dataset pack. Not used by the notebooks in this repo |

### Files created by the notebooks

| Path | Role |
|---|---|
| `PCB_USED/<class>/*_defect_XXXX.png` | Milestone 1 patches |
| `images_combined/` | All raw photos copied into one folder |
| `yolo_dataset/train|val|test` | YOLO images + `.txt` labels |
| `pcb_yolo_dataset.yaml` | Class names and split paths |
| `runs/pcb_yolo/` | Training curves, confusion matrix, `results.csv` |
| `runs/pcb_yolo/weights/best.pt` | Best training checkpoint |
| `best_yolov8_pcb.pt` | Copy used by the Streamlit app |
| `results/yolo_red_boxes/` | Inference images with red boxes |
| `final_test_evaluation_report.txt` | Test-set scores from the evaluation notebook |

---

## Streamlit demo

Notebook: `PCB-Milestone-3(Module-5 and Module-6)/Milestone-3 _ Module-5-and-Module-6.ipynb`

The app:

- Loads `best_yolov8_pcb.pt`
- Accepts `.jpg` / `.png`
- Draws thick red boxes and confidence labels
- Shows a table of detections
- Lets the user download the annotated image

It was run from Google Colab with the model path:

`/content/drive/MyDrive/PCB_DATASET/best_yolov8_pcb.pt`

To try it locally, download `best_yolov8_pcb.pt` from Drive and point `MODEL_PATH` to that file.

---

## How to read the work quickly

1. This README (problem, switch, metrics)
2. `Results/` images
3. Milestone 2 Module 3 notebook (training)
4. Milestone 2 Module 4 notebook (test numbers)
5. Drive `runs/pcb_yolo/results.csv` and `confusion_matrix.png`

---

## Limitations

- Trained and evaluated on this public aligned PCB set, not on random phone photos from a factory line
- YOLOv8n on CPU; a larger YOLO model on GPU would be the first upgrade
- Spur and mouse-bite recall is the weak point
- Milestone 1 one-way subtraction (`template - test`) is weaker for additive defects (short, spur, spurious copper). That path was dropped for the app
- Demo paths assume Google Drive / Colab unless `MODEL_PATH` is changed

---

## Stack

Python, OpenCV, Ultralytics YOLOv8, PyTorch, Pandas, Matplotlib, Seaborn, Streamlit, Google Colab

---

## What this project shows

- Built a full loop: data → labels → train → test metrics → demo
- Replaced a method that looked good on paper (patch classification ~0.90) after it failed at localization
- Report the detection metric that matches the product: **test mAP@0.5 ≈ 0.84**
- Separate easy vs hard classes instead of hiding behind one headline number
