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
![Fig. 1: Illustration of the proposed undercover bias verification scheme](/images/2026-04-17-Rethinking_Dataset_Copyright_Protection_via_Embedding_Class_wise_Hidden_Bias/fig1.png)
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
- Many prior studies have focused on protecting intellectual property (IP) in machine learning, particularly in the context of training data and models
    - Li, et al., "Untargeted backdoor watermark: Towards harmless and stealthy dataset copyright protection", NeurIPS, 2022.
    - Liu, et al., "Your model trains on my data? protecting intellectual property of training data via membership fingerprint authentication", IEEE Transactions on Information Forensics and Security 17, 1024-1037, 2022.
    - Sablayrolles, et al., "Radioactive data: tracing through training", ICML, 2020.
    - Zhang, et al., "Model watermarking for image processing networks", AAAI, 2020.
- Additionally, techniques originally developed as model attack methods (especially those involving data manipulation) can also berepurposed for dataset IP protection
    - badckdoor attacks
    - data poisoning
    - radioactive data

### 2-1. Backdoor Attacks
#### 2-1-1. Backdoor Attacks, Introduction
- Backdoor attacks aim to make a model consistently miscalssify inputs containing a hidden trigger into a predefined target class, regardless of the original content
- This is typically achieved by
    - Injecting trigger patterns into training samples
    - Modifying their labels (reffered to as the infection process)
- A large body of work focuses on designing: 
    - Less noticeable (stealthier) triggers
    - More effective attack mechanisms
- These techniques can be used not only for attacks but also for: 
    - Protecting datasets from unauthorized usage (by embedding identifiable patterns)

#### 2-1-2. Limitations of Backdoor Attacks
- Detectability issue
    - Traditional backdoor methods introduce label noise, making them detectable via visual inspection
        - Chen, et al., "Targeted backdoor attacks on deep learning systems using data poisoning", arXiv:1712.05526, 2017.
        - Gu, et al., "BadNets: Evaluating backdooring attacks on deep neural networks", IEEE Access, 2019.
        - Wang, et al., "Invisible black-box backdoor attack through frequency domain", ECCV, 2022.
- Clean-labeled backdoor approaches aim to avoid label noise: 
    - Refool: uses reflection-based natural triggers (but limited in real-world applicability)
        - Liu, et al., "Reflection backdoor: A natural backdoor attack on deep neural networks", ECCV, 2020.
    - Hidden Trigger: works mainly when fine-tuning specific layers of a reference model
        - Saha, et al., "Hidden trigger backdoor attacks", AAAI, 2020.
- Generalization challenges
    - Sleeper Agent: improves generalization via ensemble reference models and repeated retraining
        - Souri, et al., "Sleeper agent: Scalable hidden trigger backdoors for neural networks trained from scratch", NeurIPS, 2022.
    - Color Backdoor: uses color-space triggers instead of spatial patterns
        - Jiang, et al., "Color backdoor: A robust poisoning attack in color space", CVPR, 2023.
- Scalability limitation
    - Many methods struggle when applied to multiple classes simultaneously

### 2-2. Data Poisoning
- Data poisoning methods are designed to overcome the label noise issue present in traditional backdoor attacks.
- Their goal is to make a model misclassify specific benign samples into a predefined adversarial class (targeted attack).

#### 2-2-1. How Data Poisoning Works
1. Train a reference model
    - A model is first trained on a clean (benign) datset
2. Select an adversarial target class
    - Define which class the victim samples should be misclassified into
3. Choose victim samples
    - Identify specific inputs that the attacker wants to manipulate

- The poisoning process: 
    - Modify some training samples from the adversarial class
    - Use adversarial attack techniques to move them close to victim samples in latent space
- After fine-tuning: 
    - The model's decision boundary shifts
    - The region around victim samples is classified as the adversarial class -> Victim samples are likely to be misclassified

#### 2-1-2. Notable Data Poisoning Methods
- Aghakhani, et al., "Bullseye polytope: A scalable clean-label poisoning attack", EuroS&P, 2021.
- Geiping, et al., "Witches’ brew: Industrial scale data poisoning via gradient matching", ICLR, 2021.
- Huang, et al., "MetaPoison: Practical general-purpose clean-label data poisoning", NeurIPS, 2020.
- Shafahi, et al., "Poison Frogs! Targeted clean-label poisoning attacks", NeurIPS, 2018.

#### 2-2-3. Limitations of Data Poisoning
- High computational cost
    - Requires iterative adversarial optimization -> computationally expensive
- Limited verification power
    - Only affects a small number of selected samples
    - Makes it harder to use as strong evidence for dataset usage

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