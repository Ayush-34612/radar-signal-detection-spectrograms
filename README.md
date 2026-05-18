# Radar Signal Detection in Spectrograms: Ablation-Driven Analysis, Hybrid Detection Framework, and Embedded Deployment

This repository contains the official implementation, research findings, and edge deployment workflow for our paper on radar signal detection in the time-frequency domain. 

---

## 📌 Project Overview
[cite_start]Radar signal detection in spectrogram representations faces significant challenges due to low signal-to-noise ratios (SNR), structural variability, and small localized patterns[cite: 689]. [cite_start]While modern object detection architectures perform well, clear understanding of which design choices are most critical has been limited[cite: 690]. 

[cite_start]This project bridges that gap through a dual-perspective approach combining model interpretability and raw performance[cite: 691]:
1. [cite_start]**Ablation-Driven Analysis:** A systematic evaluation of a YOLOv8-based detector to isolate the exact contributions of feature hierarchy, spatial resolution, feature fusion, and model capacity[cite: 692].
2. [cite_start]**Hybrid Detection Framework:** An ensemble framework that merges complementary predictions from multiple models to optimize detection sensitivity and robustness[cite: 694, 829].
3. [cite_start]**Embedded Edge Deployment:** Practical validation of a lightweight, optimized YOLOv8 Nano model running in real-time on a resource-constrained embedded platform[cite: 696].

---

## 📊 Dataset & Preprocessing
[cite_start]Models are trained and evaluated on the **NIST-CBRS dataset** using the **MaxHold spectrogram representation** (`NISTSpecMaxHold512`)[cite: 715].
* [cite_start]**Dimensions:** Each sample is a $512\times512$ image representing radar signals as localized patterns in the time-frequency domain[cite: 719].
* [cite_start]**Classes:** Multiple radar classes including PON and Q3N variants[cite: 720].
* [cite_start]**Advantage:** The max-hold operation aggregates peak energy over time, which enhances signal visibility under low-SNR conditions and reduces temporal variability[cite: 717, 718, 826].

---

## 🔬 Key Ablation Study Findings
[cite_start]Our systematic ablation experiments revealed a clear hierarchy of importance among architectural components[cite: 750, 807]:

| Ablation Configuration | $mAP_{50}$ | Precision | Recall | $\Delta mAP_{50}$ | $\Delta Recall$ | Key Insight |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Baseline (Full YOLOv8)** | **0.9305** | **0.9527** | **0.8917** | *Ref* | *Ref* | [cite_start]Strong baseline, with room for improvement in recall[cite: 740, 755]. |
| **No P3 Branch** | 0.8610 | 0.9102 | 0.8025 | `-0.0695` | `-0.0892` | [cite_start]**Most Critical:** Confirms radar detection is fundamentally a small-object task heavily reliant on high-resolution spatial features[cite: 761, 762, 763, 765]. |
| **No P5 Branch** | 0.9289 | 0.9511 | 0.8890 | `-0.0016` | `-0.0027` | [cite_start]**Minimal Impact:** High-level global semantic features contribute very little to local structural patterns[cite: 770, 771, 772, 773]. |
| **Frozen Backbone** | 0.9030 | 0.9215 | 0.8450 | `-0.0275` | `-0.0467` | [cite_start]**Fine-Tuning Required:** Pretrained natural image features fail to generalize without domain-specific adaptation[cite: 776, 778, 779, 780, 782]. |
| **No PAN Connections** | 0.9152 | 0.9360 | 0.8653 | `-0.0153` | `-0.0264` | [cite_start]Multi-scale feature fusion provides crucial support for spatial/semantic alignment[cite: 785, 786, 789]. |
| **Half Channels** | 0.9128 | 0.9324 | 0.8611 | `-0.0177` | `-0.0306` | [cite_start]Structural design and layer arrangement are more important than raw model capacity[cite: 792, 793, 795]. |
| **Spatial Perturbation (Blur)**| 0.9021 | 0.9203 | 0.8422 | `-0.0284` | `-0.0495` | [cite_start]Removing fine structural details heavily degrades precise time-frequency boundaries[cite: 798, 799]. |
| **Noise Input (Gaussian)** | 0.8955 | 0.9158 | 0.8350 | `-0.0350` | `-0.0567` | [cite_start]Ambiguity in signal patterns causes missed detections, indicating a need for denoising[cite: 803, 804]. |

---

## 📈 Hybrid Ensemble Performance
[cite_start]To maximize reliability and minimize dangerous missed detections, we implemented a hybrid prediction fusion framework[cite: 694, 829, 832]. [cite_start]By combining the YOLOv8 baseline with a complementary detector, the ensemble achieves a vastly superior balance between sensitivity and accuracy[cite: 829, 833]:

* [cite_start]**Baseline $mAP_{50}$:** 93.05% $\rightarrow$ **Ensemble $mAP_{50}$: 94.36%** [cite: 848]
* [cite_start]**Baseline Recall:** 89.17% $\rightarrow$ **Ensemble Recall: 95.21%** [cite: 848]

> [cite_start]💡 **Takeaway:** The sharp improvement in recall (from 89.17 to 95.21) signifies a massive reduction in missed signals, which is vital for electronic intelligence and spectrum monitoring environments where missing a signal has critical implications[cite: 700, 831, 832].

---

## 🛠️ Embedded Deployment (PYNQ-Z2)
[cite_start]To prove real-world edge feasibility, we successfully ported the framework to an embedded hardware architecture[cite: 696, 905]:

* [cite_start]**Target Hardware:** PYNQ-Z2 development board powered by the Xilinx Zynq-7000 SoC (Dual-core ARM Cortex-A9 processor + FPGA fabric)[cite: 907].
* [cite_start]**Optimization Workflow:** The trained YOLOv8 Nano model was exported to ONNX format[cite: 912]. [cite_start]Input spectrograms are resized to $256\times256$ and normalized prior to running inference[cite: 913]. 
* [cite_start]**Runtime Stack:** Handled entirely offline on the ARM processor via a Jupyter Notebook control interface using `onnxruntime`, `opencv-python`, and `numpy`[cite: 908, 909, 912].
* [cite_start]**Result:** Real-time localized signal bounding boxes generated efficiently at a confidence threshold of 0.25, proving the potential for low-latency, low-power edge spectrum monitoring[cite: 914, 915, 921].

---

## 📂 Repository Structure
```text
├── src/                  # Training scripts, evaluation logic, and hybrid ensemble code
├── deployment/           # ONNX export scripts and Jupyter deployment notebooks for PYNQ-Z2
├── documents/            # Academic Project Report (PDF) and Presentation Slides (PPT)
├── .gitignore            # Spec for excluding local weights, datasets, and cache files
└── README.md             # Project landing page