# SpectraSense 

> **Real-time object detection and classification system** that identifies personal belongings on a conveyor belt and communicates the object type to the IoT system for automated sorting into the correct bin.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Goals & Objectives](#goals--objectives)
- [System Architecture](#system-architecture)
- [Classes](#classes)
- [Project Phases](#project-phases)
  - [Phase 1 — Data Collection & Preparation](#phase-1--data-collection--preparation)
  - [Phase 2 — Model Training & Optimization](#phase-2--model-training--optimization)
  - [Phase 3 — Final Testing & Deployment](#phase-3--final-testing--deployment)
- [YOLO Model Comparison](#yolo-model-comparison)
- [Iterative Improvement Log](#iterative-improvement-log)
- [Results](#results)
- [Tech Stack](#tech-stack)
- [IoT Integration](#iot-integration)
- [What's Next](#whats-next)
- [Team](#team)

---

## Project Overview

SpectraSense is an AI-powered smart sorting system designed to classify personal belongings — the kind of items people typically carry with them — and route them automatically to the correct bin using a conveyor belt controlled by an IoT system.

The concept is inspired by recycling systems, but instead of sorting by material alone, SpectraSense goes one level deeper: it identifies **what the object actually is**, determines its **identity and metal content**, then communicates that classification to the hardware for physical sorting.

A camera positioned above the conveyor belt captures each object as it passes. The AI model identifies it in real time and sends the predicted label to the IoT system via ESP. Based on the label, the conveyor and servo motors activate to deliver the object to one of four designated bins.

### The Four Sorting Bins

| Bin | Category | Objects |
|-----|----------|---------|
| 1 | **Metal Electronics** | AirPods, AirPods case, charger, headphones, mobile phone, power bank |
| 2 | **Metal Accessories** | Wrist watch, necklace, ring |
| 3 | **Metal Tools** | Key, coin, bank card |
| 4 | **Not Metal** | Wallet, ID card, pen, pencil, bottle |

---

## Problem Statement

Current automated sorting systems often stop at material recognition — distinguishing metal from plastic, for example. However, knowing the material alone is not enough for advanced logistics or security applications. A coin and a key are both metal, yet they require completely different handling. A bank card and an ID card look nearly identical yet belong to different categories.

There is a need for a **multi-layered identification system** that recognizes both the substance and the specific identity of the object — enabling smarter, more precise automated sorting that goes beyond what material sensors alone can achieve.

---

## Goals & Objectives

### 1. Recognizing Object Identity
Develop a system that accurately identifies and categorizes each object — not just its material, but its specific class — enabling precise bin assignment for every item.

### 2. Dataset Diversity & Realism
Train the model using objects captured from diverse angles, orientations, and realistic perspectives to mimic actual operating conditions on a conveyor belt environment.

### 3. Environment-Specific Optimization
Calibrate the model to account for specific camera positioning and the expected nature of items, ensuring seamless transition from training to real-time deployment.

### 4. Iterative Dataset Optimization
- **Data Refinement:** Curate high-quality samples by removing noise and redundant data
- **Label Optimization:** Merge similar classes to eliminate model confusion
- **Enhanced Robustness:** Diversify training data to cover complex real-world variations

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            FULL PIPELINE                                 │
│                                                                          │
│   Object placed        Camera captures       YOLO model detects          │
│   on conveyor    ──►   object frame     ──►  & classifies object    ──►  │
│                                               label + confidence         │
│                                                                          │
│   ──►  Label sent       IoT receives          Conveyor + servos          │
│        to ESP      ──►  label via ESP    ──►  deliver object to     ──►  │
│                                               correct bin                │
└──────────────────────────────────────────────────────────────────────────┘
```

### AI Pipeline (Step by Step)

```
Camera Frame (live feed)
          │
          ▼
  ┌───────────────┐
  │   YOLO v26n   │  ◄── Detects object & predicts class in real time
  │   Detection   │      Trained on multi-source labeled dataset
  └───────┬───────┘
          │
          │  Bounding box + Class label + Confidence score
          ▼
  ┌───────────────┐
  │  OpenCV Live  │  ◄── Draws annotations on video stream
  │    Display    │      Bounding box · Label · Confidence %
  └───────┬───────┘
          │
          ▼
     Label sent to ESP
          │
          ▼
  IoT System → Conveyor + Servos → Correct Bin
```

---

## Classes

All classes represent objects people commonly carry daily, grouped into the four sorting categories:

### 🔌 Metal Electronics
`AirPods` · `AirPods Case` · `Charger` · `Headphones` · `Mobile Phone` · `Power Bank`

### 💍 Metal Accessories
`Wrist Watch` · `Necklace` · `Ring`

### 🔑 Metal Tools
`Key` · `Coin` · `Bank Card`

### 🎒 Not Metal
`Wallet` · `ID Card` · `Pen` · `Pencil` · `Bottle`

### Full Researched Class List
During the research phase, we evaluated the following candidates before finalizing the above:

> AirPods · Headphones · AirPods Case · Mobile Phone · Smart Watch · Bank Card · ID Card · Bottle · Glasses / Sunglasses · Wallet · Pen · Pencil · Eraser · Necklace · Bracelet · Ring · Charger · Power Bank · Coin · Key · Paper · Cups

---

## Project Phases

---

### Phase 1 — Data Collection & Preparation

#### Step 1 — Research & Class Selection

Before collecting a single image, the most critical decision was **what to detect**. Not all object classes are created equal — some are too rare, some are too ambiguous, and some would not realistically appear in a conveyor belt scenario.

Our selection criteria were clear: we focused on objects that **people commonly carry with them on a daily basis** — items that would realistically appear in a camera frame in public or checkpoint environments. This included things like mobile phones, keys, wallets, bottles, and similar everyday items.

#### Step 2 — Collecting Data from Multiple Sources

With our classes defined, the next step was gathering raw image data at scale. A YOLO model learns by seeing at least hundreds of examples of each object, from different angles, lighting conditions, backgrounds, and distances. A small or narrow dataset produces a fragile model — one that works in the lab but fails in the real world.

We pulled data from multiple platforms:

| Source | Description |
|--------|-------------|
| **Roboflow** | Pre-labeled computer vision datasets with existing bounding box annotations |
| **Kaggle** | Community-shared image collections |
| **Other repositories** | Various open-source datasets for specific classes |

**Datasets collected per class:**

| Class Group | Source |
|-------------|--------|
| Phone · Smart Watch · Earphone | Kaggle / Roboflow |
| Headphone · Power Bank | Kaggle / Roboflow |
| Phone · Bank Card | Roboflow |
| ID / Bank Card (×4 datasets) | Roboflow |
| Jewelry | Roboflow |
| Sunglasses & Glasses | Kaggle / Roboflow |
| Cell Phones | Kaggle |
| Paper · Bottle · Cup | Roboflow |
| Wallet (×2 datasets) | Roboflow |
| Coin · Key · Pen | Kaggle / Roboflow |
| Charger (×2 datasets) | Roboflow |
| Various multi-class datasets | Roboflow / Kaggle |

Using multiple sources was intentional — it ensures variety in image quality, resolution, scene context, and geographic diversity.

#### Step 3 — Filtering & Labeling Images

Raw data is never ready for training. After collection, we went through a **rigorous filtering process** — removing blurry images, duplicates, mislabeled examples, and images where the object was too small or partially visible to be useful. Quality always outweighs quantity in computer vision.

All accepted images were uploaded into a **shared Roboflow workspace**, where we managed labeling, versioning, and dataset splits. For images from Kaggle or other sources without bounding box annotations, **we drew every bounding box manually** — image by image — ensuring each label was tight, accurate, and consistent across the team.

> ⚠️ Manual labeling is the most time-consuming step in this project — but it is the direct foundation of model accuracy. One consistently mislabeled class can corrupt an entire detection category.

---

### Phase 2 — Model Training & Optimization

#### Step 4 — Choose & Train YOLO Model

With a clean, labeled dataset in place, we moved into model training. We chose the **YOLO (You Only Look Once)** family for its exceptional balance between detection speed and accuracy — making it ideal for real-time applications.

**Models evaluated:**

| Model | Outcome |
|-------|---------|
| YOLO 11n | Tested — problems with Bracelet, Key, Necklace, Pen, Pencil, Sunglasses |
| YOLO 26n | ✅ **Selected as final model** |
| YOLO 26s | Tested — compared against 26n |
| YOLO 26m | Tested — compared against 26n |

**Final model: YOLO 26n**

The model was trained with parameters configured for our specific use case — including image size, batch size, number of epochs, and confidence thresholds. Training was monitored continuously to catch signs of overfitting or underfitting early.

#### Step 5 — Test with Unseen Images

Once training completed, we moved into structured evaluation. Testing is not simply running the model on random images — it requires using a **dedicated test set**: images the model has never seen during training, carefully separated before the training process began.

We evaluated the model's performance on a **per-label basis**, meaning each object class was assessed individually rather than just looking at an overall average. This approach reveals:
- Which specific classes the model handles confidently ✅
- Which ones require targeted improvement ⚠️
- Whether issues are caused by insufficient examples, visual similarity, or inconsistent labeling

#### Step 6 — Maintain & Enhance Results

Achieving strong accuracy on the first training run is rare. Model improvement is an **iterative process** — a cycle of training, evaluating, identifying weak points, making targeted changes, and re-training.

**Our benchmark: most labels must reach ≥ 89% accuracy**

When a class fell below the threshold, we had two distinct strategies:

| Strategy | Action |
|----------|--------|
| **Model-side** | Switch to a different YOLO version or adjust hyperparameters |
| **Data-side** | Add diverse images, remove ambiguous examples, combine similar sub-classes, augment existing data |

> 💡 Most accuracy gains came from better data, not a better model. When a class underperformed, the data was always the first place we looked.

---

### Phase 3 — Final Testing & Deployment

#### Step 7 — Final Data Review

Before any final deployment, we conducted a **comprehensive review of the entire dataset**. After many rounds of additions, deletions, and modifications across multiple iteration cycles, it is critical to ensure the dataset is internally consistent:

- ✅ No outdated labels remaining
- ✅ No conflicting class definitions
- ✅ Train / validation / test splits correctly balanced
- ✅ Labeling conventions applied uniformly across all classes
- ✅ Final dataset version locked in Roboflow

#### Step 8 — Live Camera Test with OpenCV

The final and most revealing test of any computer vision model is not on a static image dataset — it is **in the real world, in real time**.

We integrated our trained YOLO model with **OpenCV** — a powerful open-source computer vision library that allows us to access a live camera feed, process each frame, and display detection results in real time.

Every frame captured by the camera is passed through the model, which draws bounding boxes, class labels, and confidence scores directly on the video stream. This live test reveals how the model handles:

- Real lighting conditions and shadows
- Motion blur from a moving conveyor belt
- Partial occlusions of objects
- The natural unpredictability of a live environment

> 💡 If a model performs well on static test images but struggles on live camera, the cause is usually motion blur or insufficient variety in training data.

---

## YOLO Model Comparison

| Model | Status | Notes |
|-------|--------|-------|
| YOLO 11n | ❌ Not selected | Problems with: Bracelet, Key, Necklace, Pen, Pencil, Sunglasses |
| YOLO 26n | ✅ **Final model** | Best overall performance across all classes |
| YOLO 26s | Tested | Compared against 26n |
| YOLO 26m | Tested | Compared against 26n |

---

## Iterative Improvement Log

Each iteration addressed specific underperforming classes. Below is the full log of changes made across all training runs:

| Run | Model | Changes Made |
|-----|-------|-------------|
| 1 | YOLO 11n | Initial training — problems: Bracelet, Key, Necklace, Pen, Pencil, Sunglasses |
| 2 | YOLO 26n | Removed Bracelet · Edited ID Card class · Added new data for Key · Added Eraser |
| 3 | YOLO 26n | Combined Jewelry classes · Added new data for Key & Sunglasses |
| 4 | YOLO 26n | Added new data for Key & Sunglasses · Combined Pen & Pencil into one class |
| 5 | YOLO 26n | Added Calculator & Notebook classes |
| 6 | YOLO 26n | Improved Pens/Pencils class data |
| 7 | YOLO 26n | Problems remaining: Eraser, Key, Sunglasses |
| 8 | YOLO 26n | Removed Eraser · Problems remaining: Key, Sunglasses |

---

## Results

| Metric | Value |
|--------|-------|
| **Final model** | YOLO 26n |
| **Per-class accuracy target** | ≥ 89% for most labels |
| **Inference type** | Real-time, live camera feed via OpenCV |
| **Integration** | OpenCV display + ESP communication to IoT system |

---

## Tech Stack

| Tool / Library | Purpose |
|----------------|---------|
| **YOLO 26n** | Real-time object detection and classification |
| **OpenCV** | Live camera feed processing and bounding box annotation |
| **Roboflow** | Dataset management, labeling, versioning, augmentation, splits |
| **Python** | Training pipeline, inference scripts, ESP communication |
| **Kaggle** | Data source for image collection |

---

## IoT Integration

> 📌 *This section to be completed by the IoT team*

The AI model communicates predicted labels to the IoT system via **ESP (ESP8266 / ESP32)**.

**Communication flow:**

```
YOLO Model predicts class
          │
          │  Label + Confidence Score
          ▼
     ESP Module
          │
          ├──► Conveyor belt motor (move to correct position)
          └──► Servo motors ──► Route object into correct bin
```

**Sorting logic:**

| Predicted Class | Bin |
|-----------------|-----|
| AirPods, AirPods Case, Charger, Headphones, Mobile Phone, Power Bank | Bin 1 — Metal Electronics |
| Wrist Watch, Necklace, Ring | Bin 2 — Metal Accessories |
| Key, Coin, Bank Card | Bin 3 — Metal Tools |
| Wallet, ID Card, Pen, Pencil, Bottle | Bin 4 — Not Metal |

> *[IoT team: please add hardware specifications, component list, wiring diagram, firmware details, and ESP communication protocol here]*

---

## What's Next

Our project does not end here — it marks the beginning of a larger vision. The current system successfully detects everyday objects in real time, but there is significant room to grow in both capability and impact.

### 1. 🏷️ Add New Object Classes
Expand detection coverage by introducing additional classes beyond the current set — going through the same rigorous collection, filtering, and labeling pipeline already established.

### 2. 🖥️ Launch a User-Facing GUI
Develop a graphical interface that allows real people to try the system live — removing the lab environment and exposing the model to real-world diversity of users, objects, and conditions.

### 3. 📋 Structured User Feedback Collection
Integrate a built-in survey form after each session so users can flag misclassifications and suggest improvements — turning every interaction into a data point for the next version.

### 4. 📱 Optimize for Mobile & Embedded Devices
Compress and optimize the model using quantization and pruning techniques for deployment on Raspberry Pi, mobile phones, or embedded security modules — enabling portable and low-power applications.

### 5. 🔬 Multi-Layer Object Identification
Move beyond category recognition toward full object identity — not just "this is metal" but "this is specifically a key" — enabling smarter handling decisions at every level of the sorting pipeline.

---

## Team

### AI Team

| Name | Role | GitHub | LinkedIn |
|------|------|--------|----------|
| [Mohammed Abdelnaser]| Team Lead | [GitHub](https://github.com/abdelnasser0)  | [LinkedIn] |
| Mohammed Gamal | Member | [GitHub](https://github.com/MohamadGemy04) | [LinkedIn] |
| [Hager Ali]| Member | [GitHub](https://github.com/hager-ali191)  | [LinkedIn](www.linkedin.com/in/hager-ali-460316365) |

### IoT Team

| Name | Role | GitHub | LinkedIn |
|------|------|--------|----------|
| [Malak] | Team Lead | [GitHub]() | [LinkedIn]() |
| [Manar Ashraf]| Member | [GitHub]([https://github.com/username](https://github.com/manarelgohary0)) | [LinkedIn]() |
| Jana Kassem | Member | [GitHub]((https://github.com/janaosmaneng-cyber)) | [LinkedIn]() |
| Habiba Elsayed | Member | [GitHub](https://github.com/habibaelsayed28) | [LinkedIn]() |

### Supervisors
> *[Add supervisor names and titles here]*

### Special Thanks
> *[Add names of anyone who supported or mentored the project here]*

---

## Presented At

| Event | Details |
|-------|---------|
| 🤖 **RoboTech Fair** | *[26 / 4 , 27 / 4]* |
| 🏆 **AZEX Competition** | *[4 / 5]* |

---

<div align="center">

**SpectraSense** — Smart Object Sorting System  
AI Component &nbsp;·&nbsp; YOLO 26n · OpenCV · Roboflow · Python

</div>
