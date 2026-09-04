# ALESRD: Augmenting Long Entity-Span, Rare and Difficult

This repository contains the complete source code, experimental logs, and documentation for **ALESRD**, a specialized architecture proposed to optimize Vietnamese Biomedical Named Entity Recognition.

---

## 1. Problem Statement & Exploratory Data Analysis (EDA)

An in-depth exploratory analysis of the ViMedNER biomedical dataset reveals critical data characteristics and challenges:

* **Class Imbalance:** The background label `O` dominates the dataset, accounting for 127,233 tokens (76.15%), while entity tokens (`B-`, `I-`) represent only 39,851 tokens (23.85%).


* **Frequency Disparity:** Significant skewness exists across entity categories, notably between high-resource entities (e.g., `ten_benh`) and low-resource ones (e.g., `nguyen_nhan_benh`).


* **Span Length Complexity:** Vietnamese biomedical texts frequently feature long, complex entity spans that overlap lexically with everyday language rather than strict clinical terminology.



### Baseline Limitations

A standard baseline model utilizing `vinai/phobert-base-v2` paired with a token-level Linear classifier yields an overall F1-score of 65.16%. The baseline struggles significantly with long, rare entities.

---

## 2. Proposed Architecture (ALESRD-1.0)

Naively combining Focal Loss with a CRF layer induces gradient conflicts, as Focal Loss down-weights easy examples independently per token, disrupting the sequence-level probabilistic optimization.

To overcome this, **ALESRD-1.0** introduces a harmonized auxiliary weighting mechanism:

* **Encoder Backbone:** Leverages `vinai/phobert-base-v2` for contextualized sub-word token representations.


* **Sequence Modeling & Auxiliary Loss:** Integrates a Conditional Random Field (CRF) layer with a customized Auxiliary Loss to handle long and rare entities explicitly.


* **Multi-Dimensional Weighting:** Incorporates static class weights ($w_{freq}$, $w_{diff}$) and dynamic length-aware weights ($w_{len}$) to modulate the token-level cross-entropy loss.


* **Total Loss Formulation:**

$$\text{Loss} = \text{Loss}_{\text{CRF}} + \lambda \times (w_{len} \times w_{diff} \times w_{freq}) \times \text{Loss}_{\text{CE}}$$




---

## 3. Experimental Results

Performance comparisons across all evaluated configurations on the benchmark test set (evaluated via Strict Entity-Level metrics):

| Model | Precision | Recall | F1-Score |
| --- | --- | --- | --- |
| **Baseline (PhoBERT + Linear)** | 60.41% | 70.72% | 65.16% |
| **Focal Loss** | 66.44% | 72.91% | 69.52% |
| **Focal + CRF** | 70.32% | 72.53% | 71.41% |
| **CRF Only** | 71.68% | 72.56% | 72.12% |
| **ALESRD-1.0 (Proposed)** | **71.16%** | **74.67%** | **72.87%** |

### Key Achievements

* **Overall Performance Boost:** Achieves a 7.71% relative F1 increase over the baseline, accompanied by a +10.75% surge in Precision, effectively suppressing False Positives.


* **Breakthrough on Rare Entities:** The F1-score for `nguyen_nhan_benh` jumps by +23.13% due to tailored penalty weighting and span-aware supervision.


* **Mitigation of Span Fragmentation:** Eliminates token-level decoupling, successfully capturing long medical entities (achieving a 30.30% F1, a +9.47% gain over pure CRF on ultra-long spans).


* **Uniform Gains:** Yields steady performance enhancements ranging from 4.6% to nearly 10% across common categories.



---

## 4. Limitations & Error Analysis

* **Computational Trade-off:** Training and inference overheads are higher than the baseline due to auxiliary weight calculations and Viterbi decoding.


* **Absolute Performance Ceiling on Hard Entities:** While relative gains are substantial, absolute metrics for rare classes remain constrained by dataset noise and annotation inconsistencies.



---

## 5. References

* [1] ViMedNER Dataset: [tdtrinh11/ViMedNer](https://github.com/tdtrinh11/ViMedNer).


* [2] Pretrained Model: [vinai/phobert-base-v2](https://huggingface.co/vinai/phobert-base-v2).


* [3] Tsung-Yi Lin, Priya Goyal, Ross Girshick, Kaiming He, and Piotr Dollár, "Focal Loss for Dense Object Detection," *ICCV*, 2017.


* [4] Christian Szegedy et al., "Going Deeper with Convolutions," *CVPR*, 2015.


* [5] J. Lafferty, A. McCallum, and F. C. Pereira, "Conditional random fields: Probabilistic models for segmenting and labeling sequence data," *ICML*, 2001.



---

**Author:** Phan Xuân Thành — Student ID: 25020397
