# Radar Signal Detection in Spectrograms: Ablation-Driven Analysis, Hybrid Detection Framework, and Embedded Deployment

This repository contains the official implementation, research findings, and edge deployment workflow for our paper on radar signal detection in the time-frequency domain. 

---

## 📌 Project Overview
Radar signal detection in spectrogram representations faces significant challenges due to low signal-to-noise ratios (SNR), structural variability, and small localized patterns. While modern object detection architectures perform well, clear understanding of which design choices are most critical has been limited. 

This project bridges that gap through a dual-perspective approach combining model interpretability and raw performance:
1. **Ablation-Driven Analysis:** A systematic evaluation of a YOLOv8-based detector to isolate the exact contributions of feature hierarchy, spatial resolution, feature fusion, and model capacity.
2. **Hybrid Detection Framework:** An ensemble framework that merges complementary predictions from multiple models to optimize detection sensitivity and robustness.
3. **Embedded Edge Deployment:** Practical validation of a lightweight, optimized YOLOv8 Nano model running in real-time on a resource-constrained embedded platform.

---

## 📊 Dataset & Preprocessing
Models are trained and evaluated on the **NIST-CBRS dataset** using the **MaxHold spectrogram representation** (`NISTSpecMaxHold512`).
* **Dimensions:** Each sample is a $512\times512$ image representing radar signals as localized patterns in the time-frequency domain.
* **Classes:** Multiple radar classes including PON and Q3N variants.
* **Advantage:** The max-hold operation aggregates peak energy over time, which enhances signal visibility under low-SNR conditions and reduces temporal variability.

---

## 🔬 Key Ablation Study Findings
Our systematic ablation experiments revealed a clear hierarchy of importance among architectural components:

| Ablation Configuration | $mAP_{50}$ | Precision | Recall | $\Delta mAP_{50}$ | $\Delta Recall$ | Key Insight |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Baseline (Full YOLOv8)** | **0.9305** | **0.9527** | **0.8917** | *Ref* | *Ref* | Strong baseline, with room for improvement in recall. |
| **No P3 Branch** | 0.8610 | 0.9102 | 0.8025 | `-0.0695` | `-0.0892` | **Most Critical:** Confirms radar detection is fundamentally a small-object task heavily reliant on high-resolution spatial features. |
| **No P5 Branch** | 0.9289 | 0.9511 | 0.8890 | `-0.0016` | `-0.0027` | **Minimal Impact:** High-level global semantic features contribute very little to local structural patterns. |
| **Frozen Backbone** | 0.9030 | 0.9215 | 0.8450 | `-0.0275` | `-0.0467` | **Fine-Tuning Required:** Pretrained natural image features fail to generalize without domain-specific adaptation. |
| **No PAN Connections** | 0.9152 | 0.9360 | 0.8653 | `-0.0153` | `-0.0264` | Multi-scale feature fusion provides crucial support for spatial/semantic alignment. |
| **Half Channels** | 0.9128 | 0.9324 | 0.8611 | `-0.0177` | `-0.0306` | Structural design and layer arrangement are more important than raw model capacity. |
| **Spatial Perturbation (Blur)**| 0.9021 | 0.9203 | 0.8422 | `-0.0284` | `-0.0495` | Removing fine structural details heavily degrades precise time-frequency boundaries. |
| **Noise Input (Gaussian)** | 0.8955 | 0.9158 | 0.8350 | `-0.0350` | `-0.0567` | Ambiguity in signal patterns causes missed detections, indicating a need for denoising. |

---

## 📈 Hybrid Ensemble Performance
To maximize reliability and minimize dangerous missed detections, we implemented a hybrid prediction fusion framework. By combining the YOLOv8 baseline with a complementary detector, the ensemble achieves a vastly superior balance between sensitivity and accuracy:

* **Baseline $mAP_{50}$:** 93.05% $\rightarrow$ **Ensemble $mAP_{50}$: 94.36%**
* **Baseline Recall:** 89.17% $\rightarrow$ **Ensemble Recall: 95.21%**

> 💡 **Takeaway:** The sharp improvement in recall (from 89.17 to 95.21) signifies a massive reduction in missed signals, which is vital for electronic intelligence and spectrum monitoring environments where missing a signal has critical implications.

---

## 🛠️ Embedded Deployment (PYNQ-Z2)
To prove real-world edge feasibility, we successfully ported the framework to an embedded hardware architecture:

* **Target Hardware:** PYNQ-Z2 development board powered by the Xilinx Zynq-7000 SoC (Dual-core ARM Cortex-A9 processor + FPGA fabric).
* **Optimization Workflow:** The trained YOLOv8 Nano model was exported to ONNX format. Input spectrograms are resized to $256\times256$ and normalized prior to running inference. 
* **Runtime Stack:** Handled entirely offline on the ARM processor via a Jupyter Notebook control interface using `onnxruntime`, `opencv-python`, and `numpy`.
* **Result:** Real-time localized signal bounding boxes generated efficiently at a confidence threshold of 0.25, proving the potential for low-latency, low-power edge spectrum monitoring.

---

## 📂 Repository Structure
```text
├── src/                  # Training scripts, evaluation logic, and hybrid ensemble code
├── deployment/           # ONNX export scripts and Jupyter deployment notebooks for PYNQ-Z2
├── documents/            # Academic Project Report (PDF) and Presentation Slides (PPT)
├── .gitignore            # Spec for excluding local weights, datasets, and cache files
└── README.md             # Project landing page