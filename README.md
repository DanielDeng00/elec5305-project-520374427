# ELEC5305 Project Proposal  

## 📌 Project Title  
**Deep Learning-Based Audio Quality Restoration for Remote Speech Transmission**  

---
## 📌 Student Information
Name: Ruiyi Deng

Student ID: 520374427

GitHub Username: DanielDeng00

GitHub Project Link: https://github.com/DanielDeng00/elec5305-project-520374427.git

---

## 📝 Project Overview  
Remote speech transmission is highly susceptible to degradations such as **background noise, packet loss, echo, and channel distortion**, often yielding speech that is muffled, distorted, or hard to understand. This affects everyday communication and poses challenges to **remote education, telemedicine, and collaborative work**.  

This project develops a **speech restoration system** using modern deep learning. We target recovery of speech degraded during long-distance transmission. By employing models such as **convolutional autoencoders** and **Transformer/Conformer-based architectures**, we aim to reconstruct **intelligible, perceptually natural** speech that is closer to the original signal.

---

## 📦 Data Source  
We use the **VoiceBank-DEMAND** dataset hosted by the University of Edinburgh’s DataShare (link: <https://datashare.ed.ac.uk/handle/10283/2791>). It combines clean utterances (VoiceBank) with **DEMAND** environmental noises to produce noisy speech across varied SNRs and scenes, and is widely used for speech restoration research. A common configuration uses ~28 speakers for training and 2 for testing with **non-overlapping noises/SNRs** to assess generalization.  
The official release is **48 kHz**; in practice we **resample to 16 kHz**, normalize amplitude, and segment audio as needed. During training, we **stochastically superimpose remote-transmission degradations**—including packet loss (with PLC variants), echo/residual reverberation, bandwidth limiting and channel magnitude–frequency distortion, and common codec artifacts—to better approximate real-world conditions.

---

## 🎯 Background and Motivation  
Classical speech enhancement (e.g., **spectral subtraction, Wiener filtering, statistical noise modeling**) can reduce simple additive noise, but often struggles with **nonlinear, structural** degradations from channel fading, packet loss, and bandwidth limits.  

Deep learning has transformed the field: convolutional/recurrent networks learn **time–frequency structures** and recover corrupted components; more recently, **Transformer-based architectures** capture long-term dependencies in audio [1]. This creates an opportunity to design systems tailored to **remote transmission** challenges, where degradations are not merely additive.

---

## ⚙️ Proposed Methodology  
This project is implemented entirely in **Python**, enabling a cohesive workflow for signal processing, baselines, training, visualization, and evaluation.

1. **Signal Processing**
   - Use **Short-Time Fourier Transform (STFT)** [2] to obtain spectrograms.
   - Implement **Wiener filtering** as a classical baseline within a unified evaluation pipeline.

2. **Neural Models**
   - **Phase 1**: A **U-Net–style convolutional autoencoder** estimates time–frequency (T–F) masks on **log-magnitude** spectra.
   - **Phase 2**: Explore **Transformer/Conformer** blocks for modeling long-range dependencies [1], maintaining drop-in compatibility.

3. **Training & Inference**
   - Loss: **SI-SDR** (primary) + **log-spectral MSE**, with gradient clipping and spatial padding/alignment to stabilize training.
   - Reconstruction via **iSTFT** using estimated magnitude and noisy phase (with extensions planned for phase-aware/complex masks).

4. **Evaluation**
   - Visualizations: **waveforms** and **log-power spectrograms**.
   - Objective metrics (no external toolboxes required): **SI-SDR**, **Segmental SNR (20 ms)**, **LSD**, **Mel-LSD**, **Spectral Convergence**.
   - Side-by-side reporting for **Noisy → Denoised → Clean**.

---

## 🧪 Methodology Progress  
We have completed an **end-to-end Python prototype**. On the data side, we implemented **multi-source degradation simulation** for remote scenarios: additive background noise, packet loss (including burst losses and PLC strategies such as muting, zero-filling, and frame-hold), echo/residual reverberation, bandwidth limiting and channel magnitude–frequency distortion, and common codec artifacts. These degradations are **stochastically combined online** during training to better match real-world network speech.  
For representation, we use **STFT** and feed the **log-magnitude spectrogram** to a **U-Net–style** model to estimate a T–F mask; **iSTFT** reconstructs waveforms from the estimated magnitude with noisy phase. The training pipeline employs **SI-SDR + log-spectral MSE**, gradient clipping, and mask padding/alignment for stability. Our evaluation suite includes waveform/spectrogram plots and objective metrics (**SI-SDR, Segmental SNR, LSD, Mel-LSD, Spectral Convergence**) for **Noisy → Denoised → Clean** comparisons.  
For classical baselines, **STFT and noise-estimation interfaces** are in place and **Wiener filtering** will be run in the same evaluation pipeline. We have also **reserved interfaces/data feeders** for **Transformer/Conformer** modules, enabling fast swaps and reproducible experiments.

---
  
## 🎯Stage-1 and Stage-2 Code Workflow

The two-stage speech enhancement system consists of Stage-1 speech restoration and Stage-2 spatialization. Stage-1 reconstructs clean monaural speech from remote-channel degraded audio, and Stage-2 transforms the restored signal into perceptually natural stereo. Both stages are implemented with modular code components that cover data preparation, model definition, training, evaluation, and visualization.

### Stage-1: Remote-Channel Speech Restoration
Stage-1 begins by standardizing audio input to 16 kHz and ensuring compatibility between different library versions. A remote-channel degradation module generates realistic noisy speech by applying bandwidth limitation, codec artifacts, colored noise, shuffled-speech interference, echo, and burst packet loss. Clean targets are refined through a perceptual enhancer that applies A-weighted loudness shaping, formant emphasis, and ERB smoothing. The dataset loader pairs noisy and clean audio and performs random segment extraction to increase data diversity.

A compact Transformer model takes log-magnitude STFT features and predicts a spectral mask to recover the clean magnitude. Training incorporates spectral MSE, time-domain L1, SI-SDR, and mask smoothness losses. Validation is performed after each epoch, and visualization tools display waveform comparisons, spectrograms, and objective metrics to demonstrate restoration performance.

### Stage-2: Mono-to-Stereo Spatialization
Stage-2 uses the enhanced monaural speech as input and generates stereo output. A clean-to-stereo teacher dataset is constructed by passing clean audio through a stereo teacher model. A unified STFT front end converts monaural signals into log-magnitude spectrograms, and a consistent ISTFT front end ensures accurate reconstruction.

The ResFCSpatializer model applies frame-wise residual MLP blocks to predict left and right magnitude masks, controlling ILD. Phase cues are not learned; instead, a DSP-based phase synthesis module generates IPD and ITD. This module estimates or defines a time delay τ(t), converts it into frequency-dependent phase shifts, and applies smoothing. Low-frequency regions receive stronger phase rotation to match psychoacoustic properties.

Training uses spectral MSE, downmix SI-SDR, and ILD loss. The final stereo audio is reconstructed by combining network-predicted magnitudes with DSP-generated phase. Evaluation includes ILD alignment, IPD deviation, IACC behavior, and downmix SI-SDR, with plotting tools for visualization. Together, Stage-2 blends learned magnitude cues with deterministic phase control to produce natural and interpretable stereo output.

---

## 📈 Outcomes Progress  
Under cross-sample evaluation, the system shows **stable, substantial improvements** at the dataset level. The reconstructed spectrograms align more closely with references, with the **background noise floor effectively suppressed**. Consistency across **objective metrics** and **visualizations** supports a cleaner, more natural subjective impression and indicates **robust generalization** across diverse samples.  
Analysis also reveals **areas for refinement**: a subset of cases exhibits mild **high-frequency over-suppression/over-smoothing**, motivating enhancements to **loss design** and **mask constraints**. We are prioritizing stronger **high-frequency fidelity**, **multi-resolution spectral losses**, and **phase-aware/complex-mask** modeling. We will also perform **grouped and distributional analyses** by degradation type/intensity and integrate **Wiener filtering** as a classical baseline for direct comparison. Where feasible, we will add **PESQ/STOI** or no-external-library proxies to further substantiate perceptual gains.

<h3 align="center">Dataset Mean (Noisy vs. Denoised)</h3>
<p align="center">
  <img width="398" height="395" alt="image" src="https://github.com/user-attachments/assets/64e15cba-af58-4f04-8deb-aedb3bddf30d" />
</p>

<h3 align="center">Dataset Std (Noisy vs. Denoised)</h3>
<p align="center">
  <img width="404" height="412" alt="image" src="https://github.com/user-attachments/assets/59738895-20a9-47e5-abd7-6c2d79bce950" />
</p>

<h3 align="center">Improvements (Denoised − Noisy) Mean</h3>
<p align="center">
  <img width="413" height="331" alt="image" src="https://github.com/user-attachments/assets/7f37e681-72c4-46a6-9b75-8a4fc9b9a7c9" />
</p>

<h3 align="center">Improvements (Denoised − Noisy) Std</h3>
<p align="center">
  <img width="424" height="338" alt="image" src="https://github.com/user-attachments/assets/c038ad69-91d9-4392-8b3a-ef0476014c3f" />
</p>

<h3 align="center">Objective Metrics — Dataset Mean ± Std (no PESQ/STOI)</h3>
<p align="center">
  <img width="615" height="371" alt="image" src="https://github.com/user-attachments/assets/915abb69-6efa-4b29-8b67-17918faf8c6d" />
</p>


---

## ⏱️ Timeline (Weeks 5–13)

| Weeks       | Tasks                                                                 |
|-------------|------------------------------------------------------------------------|
| **5–7**     | Literature review; dataset preparation; baseline Wiener setup.         |
| **8–9**     | Implement U-Net–style model; preliminary feasibility tests.            |
| **10–11**   | Optimization & evaluation; explore Transformer/Conformer variants.     |
| **12–13**   | Final report; GitHub docs; audio demos and release packaging.          |

---

## ✅ Conclusion  
This project addresses the **multi-source degradations** of remote speech transmission with a prototype that unifies **traditional signal processing** and **deep learning**. The current system demonstrates **clear dataset-level effectiveness** and a consistent trend toward **cleaner, more natural** speech, supported by objective metrics and visual validation. Ongoing work focuses on preserving **high-frequency detail**, broadening **grouped/distributional evaluations**, and integrating **classical baselines** and **perceptual indicators** for completeness. With continued refinements, we anticipate further gains in **clarity** and **naturalness**, alongside reproducible resources of practical value to research and industry.

---

## 📚 References  
[1] Yu, W., et al. *SETransformer: Speech Enhancement Transformer.* Cognitive Computation 14(3), 2022.  
[2] Wang, Z.-Q., et al. *STFT-domain Neural Speech Enhancement with Very Low Algorithmic Latency.* IEEE/ACM TASLP 31, 2022.  
[3] Oruh, J., & Viriri, S. *Spectral Analysis for Automatic Speech Recognition and Enhancement.* Proc. ICMlN, 2020.

