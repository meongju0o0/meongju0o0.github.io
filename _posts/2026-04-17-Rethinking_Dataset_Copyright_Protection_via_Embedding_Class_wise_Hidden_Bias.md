---
layout: single
title:  "Rethinking Data Bias: Dataset Copyright Protection via Embedding Class-wise Hidden Bias"
categories: PaperReview
tag: [deep_learning, paper_review]
toc: true
toc_sticky: true
author_profile: true
---

# Rethinking Data Bias: Dataset Copyright Protection via Embedding Class-wise Hidden Bias
## 논문 정보
- Jinhyeok Jang, BungOk Han, Jaehong Kim, Chan-Hyun Youn
- Computer Vision - ECCV 2024
- ETRI (Electronics and Telecommunications Research Institute) / KAIST (Korea Advanced Institute of Science and Technology)

## 1. Introduction
### Background
- Over the past decade, data-driven AI using deep neural networks (DNNs) has advanced significantly
- Public datasets have been a key driver by: 
    - Providing large-scale training data
    - Enabling transparent and objective benchmarking
- Representative datasets: 
    - ImageNet, MNIST, CIFAR10, Pascal VOC, MS-COCO

### Problem
- Most public datasets are restricted to: 
    - Non-commercial and educational use
    - Commercial and requires permission or fees
- In practice, violations still occur: 
    - Unauthorized commercial usage
    - Cheating in competitions (e.g., training on test data)
- Real-world cases show this is a persistent issue
- Core challenge -> **Detecting and proving unauthorized dataset usage is difficult**

### Challenge in Black-box Setting
- Realistic scenario: black-box access only
    - Available: input -> predicted class
    - Not available: architecture, weights, logits
- Therefore: 
    - Verification must rely solely on input-output behavior
    - Requires strong, output-based evidence

### Proposed Idea: Undercover Bias
- Key observation: 
    - DNNs learn not only task-relevant features but also hidden data biases
    - Models can even operate using bias-only information
- Existing works: 
    - Focus on removing bias for fairness
- This paper: 
    - Intentionally embeds class-wise hidden bias
    - Uses it as a dataset watermark

### Method Overview
[Fig. 1: Illustration of the proposed undercover bias verification scheme](/images/2026-04-17-Rethinking_Dataset_Copyright_Protection_via_Embedding_Class_wise_Hidden_Bias/fig1.png)
- Use an auxiliary dataset to generate:
    - Class-wise, undetectable hidden biases (watermarks)
- Embed these into the target dataset
- Result
    - Models trained on this data learn both: 
        - Task features
        - Hidden biases
- Verification
    - Input bias-only samples
    - If the model classifies them correctly -> evidence of dataset usage

### Contributions
1. A novel verification method based on hidden bias classification
2. Clean-labeled, model-agnostic watermarking approach
3. Extensive experimental validation agianst prior methods
4. Strong generalization across datasets, architectures, and tasks

## 2. Related Work
### 2-1. Backdoor Attacks
### 2-2. Data Poisoning
### 2-3. Radioactive Data
### 2-4. Limitations of the Prior Works in Verification

## 3. Motivation

## 4. Method
### 4-1. Noise Patch Placement: Class-wise Bias Embedding
### 4-2. Overlaying Auxiliary Dataset: Robust Bias to Augmentation
### 4-3. Undercover Bias: Invisible Bias Embedding
### 4-4. Discussion

## 5. Experiments I: Comparison with Prior Works
### 5-1. Comparison in Fundamental Specifications
### 5-2. Comparison in Effectiveness of Watermark

## 6. Experiments II: General Applicability
### 6-1. Application to Further Architectures and Datasets
### 6-2. Application to Fine-grained Classification
### 6-3. Application to Image Segmentation

## 7. Ablation Studies
### 7-1. Histogram Analysis of mAcc on Watermark
### 7-2. Visualizations

## 8. Conclusion