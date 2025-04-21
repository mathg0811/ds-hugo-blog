+++
date = '2025-04-21T22:19:00+09:00'  # 작성 날짜
draft = false  # 초안 여부
title = 'CV Features'  # 글 제목
tags = ['computer vision', 'machine learning', 'feature']  # 태그 목록
categories = ['CV', 'ML']  # 카테고리 목록
url = '/posts/feature/'  # 커스텀 URL 경로
description = 'computer vision features'  # 페이지 설명
images = ['/images/cover.jpg']  # 대표 이미지 
# authorbox= false
# sidebar= false
# pager= false
# weight= 1
menus = 'main'
+++

# Feature Description

## Feature in Data structures

- Numerical features
- Categorical features
- Image features

## Good Descriptor

- Invariant to transformation
- Diustinctiveness
- Dimensionality
- Locality
- Repeatability
- Compatibility with Matching Algorithms
- Computational Efficiency
- Adaptability
- Noise Robustness

## Feature Descriptor Techniques

---

### 1. Traditional Feature Descriptors

#### **SIFT (Scale-Invariant Feature Transform)**

![Basic Working of SIFT](https://huggingface.co/datasets/hf-vision/course-assets/resolve/main/feature-extraction-feature-matching/Original-SIFT-algorithm-flow.png)

- **Purpose:** Scale/Rotation-invariant feature detection/description  
- **Core Structure:**  
  - DoG (Difference of Gaussian) for multi-scale keypoint detection  
  - 128D orientation histogram around keypoints  
- **Features:** Robust to lighting/rotation/scale changes. Slow speed.  
- **OpenCV Support:**  
~~~python
import cv2
sift = cv2.SIFT_create()
kp, des = sift.detectAndCompute(img, None)
~~~

---

#### **SURF (Speeded-Up Robust Features)**
- **Purpose:** Faster alternative to SIFT for real-time applications  
- **Core Structure:**  
  - Hessian matrix-based keypoint detection  
  - 64/128D descriptor using Haar wavelet responses  
- **Features:** 3x faster than SIFT. Patent-restricted.  
- **OpenCV Support:**  
~~~python
surf = cv2.xfeatures2d.SURF_create()
kp, des = surf.detectAndCompute(img, None)
~~~

---

#### **BRIEF (Binary Robust Independent Elementary Features)**
- **Purpose:** Ultra-fast binary descriptor  
- **Core Structure:**  
  - Compares 256 pixel pairs → 256-bit binary vector  
- **Features:** Extremely fast with low memory usage. No rotation invariance.  
- **OpenCV Support:**  
~~~python
brief = cv2.xfeatures2d.BriefDescriptorExtractor_create()
kp, des = brief.compute(img, kp)
~~~

---

#### **ORB (Oriented FAST and Rotated BRIEF)**
- **Purpose:** Patent-free real-time alternative  
- **Core Structure:**  
  - FAST corner detection + Harris response  
  - Rotation-aware rBRIEF descriptor  
- **Features:** Best for mobile/embedded systems (100+ FPS).  
- **OpenCV Support:**  
~~~python
orb = cv2.ORB_create()
kp, des = orb.detectAndCompute(img, None)
~~~

---

#### **BRISK (Binary Robust Invariant Scalable Keypoints)**
- **Purpose:** Scale-invariant binary descriptor  
- **Core Structure:**  
  - AGAST corner detector  
  - Radial sampling pattern (512-bit)  
- **Features:** Better scale invariance than ORB.  
- **OpenCV Support:**  
~~~python
brisk = cv2.BRISK_create()
kp, des = brisk.detectAndCompute(img, None)
~~~

---

#### **FLANN (Fast Library for Approximate Nearest Neighbors)**
- **Purpose:** High-speed feature matching  
- **Core Structure:**  
  - KD-Tree/LSH algorithms  
- **Features:** Essential for large-scale matching.  
- **OpenCV Support:**  
~~~python
flann = cv2.FlannBasedMatcher(index_params, search_params)
matches = flann.knnMatch(des1, des2, k=2)
~~~

---

### 2. Neural Network-Based Descriptors

#### **SuperPoint**
- **Purpose:** Joint keypoint detection + description  
- **Core Structure:**  
  - CNN outputs heatmap + descriptor map  
- **Features:** 30 FPS real-time performance.  
- **PyTorch (Kornia) Support:**  
~~~python
from kornia.feature import SuperPoint
model = SuperPoint()
out = model(img)
keypoints, descriptors = out.keypoints, out.descriptors
~~~

---

#### **D2-Net**
- **Purpose:** Unified detection/description network  
- **Core Structure:**  
  - CNN with dense feature extraction  
- **Features:** Superior in challenging lighting/viewpoints.  
- **PyTorch Implementation:**  
~~~python
# See official repo: https://github.com/mihaidusmanu/d2-net
~~~

---

#### **GNN/Transformer-Based Methods**
- **Purpose:** Context-aware feature matching  
- **Core Structure:**  
  - Graph Neural Networks / Cross-Attention Transformers  
- **Features:** SOTA in 3D matching and complex scenes.  
- **Framework Support:**  
~~~python
# PyTorch Geometric / HuggingFace Transformers
~~~

---

### 3. Performance Metrics

| Metric            | Description                                  | Top Performers |
|-------------------|----------------------------------------------|----------------|
| **Repeatability** | Feature redetection rate                     | SIFT > SuperPoint |
| **Matching Score**| Correct match ratio                          | SuperPoint > D2-Net |
| **Latency**       | Processing time (640x480)                    | ORB(15ms) < SuperPoint(33ms) |
| **Descriptor Size** | Memory per feature                        | BRIEF(32B) < ORB(32B) < SuperPoint(256B) |

---

### 4. Evolution Timeline

1. **SIFT/SURF** (1999-2006): Accuracy-focused  
2. **BRIEF/ORB/BRISK** (2010-2011): Speed-optimized  
3. **FLANN** (2009): Large-scale matching  
4. **SuperPoint/D2-Net** (2018-2019): Deep learning revolution  
5. **GNN/Transformers** (2020-): Context-aware matching  

---

### 5. Selection Guide

- **Real-Time/Mobile:** ORB/BRISK + FLANN  
- **Accuracy:** SIFT/SuperPoint/D2-Net  
- **3D/Complex Scenes:** GNN/Transformer methods  
- **Big Data:** FLANN + D2-Net  


