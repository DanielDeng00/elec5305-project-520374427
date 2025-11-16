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

This repository implements a two-stage deep learning pipeline for **restoring** and **spatializing** speech degraded during long-distance transmission. Real communication channels introduce complex distortions—such as **bandwidth restriction, codec artifacts, echo, colored noise, and packet loss**—which significantly reduce intelligibility and perceptual quality.

### Stage 1 — Remote Speech Restoration  
Stage 1 uses a **lightweight Transformer-based denoising model** operating in the STFT magnitude domain.  
A custom **RemoteChannelAugmentor** simulates realistic remote-channel impairments, including:
- low-bitrate codec and µ-law artifacts  
- narrowband filtering  
- multi-tap echo and residual reverberation  
- colored background noise and speechbleed  
- random + burst packet loss with basic PLC reconstruction  

The network is trained with spectral and time-domain losses (log-MSE, L1, SI-SDR), producing a clean and perceptually natural **mono** signal suitable for downstream processing.

### Stage 2 — Neural-Classical Hybrid Spatialization  
Stage 2 reconstructs spatial cues and generates **binaural stereo** audio from the restored mono signal.  
The system adopts a hybrid approach:
- The neural network predicts **interaural level differences (ILD)**.  
- **Interaural time/phase differences (ITD/IPD)** are synthesized using DSP techniques:
  - GCC-PHAT delay estimation  
  - low-frequency phase regression  
  - frequency-dependent IPD modulation  
  - smoothing and energy-protection filters  

This design avoids unstable learned-phase models and ensures physically coherent, realistic spatialization.

### Outcome  
The full pipeline transforms degraded input into clean, intelligible, and spatially immersive stereo audio.  
Its modular neural-DSP design makes it suitable for:
- remote communication and VoIP  
- conferencing and telepresence  
- accessibility tools  
- multimedia and interactive audio applications

---

## 📦 Data Source  
We use the **VoiceBank-DEMAND** dataset hosted by the University of Edinburgh’s DataShare (link: <https://datashare.ed.ac.uk/handle/10283/2791>). It combines clean utterances (VoiceBank) with **DEMAND** environmental noises to produce noisy speech across varied SNRs and scenes, and is widely used for speech restoration research. A common configuration uses ~28 speakers for training and 2 for testing with **non-overlapping noises/SNRs** to assess generalization.  
The official release is **48 kHz**; in practice we **resample to 16 kHz**, normalize amplitude, and segment audio as needed. During training, we **stochastically superimpose remote-transmission degradations**—including packet loss (with PLC variants), echo/residual reverberation, bandwidth limiting and channel magnitude–frequency distortion, and common codec artifacts—to better approximate real-world conditions.

---

## 🎯 Background and Motivation  
Classical speech enhancement (e.g., **spectral subtraction, Wiener filtering, statistical noise modeling**) can reduce simple additive noise, but often struggles with **nonlinear, structural** degradations from channel fading, packet loss, and bandwidth limits.  

Deep learning has transformed the field: convolutional/recurrent networks learn **time–frequency structures** and recover corrupted components; more recently, **Transformer-based architectures** capture long-term dependencies in audio [1]. This creates an opportunity to design systems tailored to **remote transmission** challenges, where degradations are not merely additive.

---

## ⚙️ Methodology

This project adopts a two-stage deep learning framework that restores long-distance
transmission–degraded speech and subsequently generates a spatially coherent stereo output.
Both stages share a unified STFT front end and are implemented entirely in Python using
PyTorch to maintain consistency across preprocessing, augmentation, training, and evaluation.

### Stage 1 — Remote Speech Restoration
Clean speech from the VoiceBank-DEMAND dataset is transformed into realistic remote-channel
degraded signals using a custom **RemoteChannelAugmentor**. The augmentation pipeline
introduces:

- bandwidth limitation and low-rate codec distortion  
- µ-law companding and resampling artifacts  
- multi-tap echo and residual reverberation  
- colored noise and shuffled-speech leakage  
- frame-level random and burst packet loss with PLC-style reconstruction  

The degraded waveform is converted to **log-magnitude STFT features**[2], and a lightweight
**Transformer-based model**[1] predicts a magnitude mask. The enhanced magnitude is combined
with the noisy phase and reconstructed using iSTFT. Training minimizes a multi-term loss:

- log-spectral MSE  
- SI-SDR  
- time-domain L1 loss  
- mask smoothness regularization  

The target signal is a **perceptually enhanced mono reference** obtained through A-weighting,
ERB smoothing, and high-frequency presence boosting.

### Stage 2 — Neural–Classical Hybrid Spatialization
Stage 2 takes the restored mono waveform and produces binaural stereo audio. Instead of
learning complex phase, the system uses a **hybrid neural–DSP design**:

- A **ResFC (residual MLP)** predicts left/right magnitude masks encoding ILD.
- **ITD/IPD** cues are extracted from a stereo teacher using:
  - GCC-PHAT delay estimation  
  - low-frequency phase regression  
  - frequency-dependent phase weighting  
  - temporal EMA smoothing  

The predicted magnitudes and DSP-generated phase cues form stereo STFTs, which are
reconstructed via iSTFT. Training optimizes:

- log-spectral MSE  
- downmix SI-SDR  
- frequency-band ILD loss  

Evaluation includes ILD, IPD, IACC, and objective quality metrics to verify spatial fidelity
and ensure the stereo output remains consistent with the teacher in both energy distribution
and phase structure.

### Summary
Stage 1 restores intelligible mono speech under severe degradations, and Stage 2 reconstructs
realistic spatial cues using a physically interpretable stereo synthesis approach. Together, the
system converts harsh remote-channel speech into high-quality, natural, and spatially immersive
audio.

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

The updated results show that the proposed two-stage system provides strong and stable
performance across both denoising and spatialization tasks.

### Stage 1 — Speech Restoration
Across the four evaluation samples, the Transformer-based denoiser achieves:

- **SI-SDR improvements** of **+5.96 to +7.44 dB** (avg. **+6.95 dB**)  
- **Segmental-SNR gains** of **+9.00 to +13.21 dB**  
- **Large reductions** in LSD, mel-LSD, and SpecConv  
- **Strong negative MCD shifts** (−138 to −213 dB), indicating accurate recovery of formants  
- **Major outperformance** over the classical Wiener baseline (**+1.17 dB SI-SDR**)

Waveform and spectrogram comparisons show significant noise-floor suppression, clearer
harmonics, and restored transient structure. The denoised outputs visually converge toward
clean targets while avoiding artifacts or spectral holes.

<h3 align="center">Waveform and Spectrogram Comparison for Denoised Speech(sample)</h3>
<p align="center">
  <img width="1227" height="954" alt="P1" src="https://github.com/user-attachments/assets/0487102f-a7ee-4a3c-88ee-7117815b61f5" />
</p>

<h3 align="center">Lightweight Transformer Noise Reduction Data Analysis Comparison Results</h3>
<p align="center">
  <img width="1214" height="312" alt="P1-1" src="https://github.com/user-attachments/assets/0148f79f-ff30-45f1-8dd0-9f66ec8fe841" />
</p>

### Stage 2 — Spatialization
The hybrid neural–DSP spatialization yields physically coherent stereo output:

- **ILD_RMSE**: 0.86–1.20 dB  
- **IPD_L1**: 1.44–1.47 rad  
- **ΔIACC(pred–tgt)**: −0.053 to −0.130, indicating close spatial coherence  
- **ΔSI-SDR**: −2.31 to −14.88 dB (expected for mono → stereo conversion)

Frequency-domain comparisons confirm that ILD and IPD curves closely follow the teacher
stereo patterns. IACC-per-frame plots show stable coherence, with small deviations during
high-energy formant transitions.

<h3 align="center">Frequency-Domain Spatialization Analysis (ILD, IPD, and IACC)(sample)</h3>
<p align="center">
  <img width="1692" height="759" alt="P2" src="https://github.com/user-attachments/assets/0989402e-05c0-4250-8ce4-8bc7fd4b0e59" />
</p>

<h3 align="center">Lightweight ResFCS Spatialization Data Analysis Comparison Results</h3>
<p align="center">
  <img width="1737" height="456" alt="P2-1" src="https://github.com/user-attachments/assets/787e58ab-7e1c-4e5a-b124-3fdf323a9dc2" />
</p>

### Summary
Overall, the system reliably restores degraded speech and generates spatially meaningful
stereo audio. Minor limitations remain—such as high-frequency over-smoothing and narrow-band
ILD fluctuations—pointing toward future improvements using multi-resolution spectral losses,
frequency-dependent constraints, and refined phase modeling.


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
This project introduces a two-stage framework for restoring and spatializing speech degraded
by long-distance transmission. By combining lightweight Transformer-based speech restoration
with classical DSP-driven stereo cue reconstruction, the system achieves both **high-quality
denoising** and **physically consistent spatialization**.

In Stage 1, the Transformer model demonstrates robust performance across diverse channel
impairments—including broadband noise, narrowband interference, codec-style distortions, and
mixed degradation patterns. The restored speech exhibits a cleaner temporal envelope, clearer
harmonic structure, and improved spectral readability. Objective evaluations (SI-SDR,
Segmental-SNR, LSD, mLSD, MCD, SpecConv) show **consistent and substantial improvements**,
confirming the model’s ability to suppress non-speech energy while preserving linguistically
important details.

Stage 2 operates on the cleaned mono signal and reconstructs binaural cues by combining
neural magnitude estimation with classical ITD/IPD-based phase synthesis. The resulting stereo
signals exhibit **accurate ILD patterns, plausible low-frequency phase behavior, and stable
interaural correlation**, closely matching the reference stereo. Spatialization metrics across
samples validate the reliability of the stereo reconstruction. The expected SI-SDR drop during
mono-to-stereo conversion reflects the intentional introduction of interaural differences rather
than a loss of perceptual quality.

Together, these findings show that the proposed framework effectively unifies **deep neural
representations** with **interpretable, physically grounded signal processing**. The system is
capable of converting severely degraded remote speech into **clean, natural, and perceptually
stable binaural audio**, offering a scalable and computationally efficient solution for remote
communication, telepresence, assistive listening, virtual reality audio, and other immersive
applications.

Future work will explore **higher-resolution spatial synthesis**, **cross-domain stereo transfer
learning**, and **perceptually informed optimization**, building on the foundation established
by this prototype.

---

## 📚 References  
[1] Yu, W., et al. *SETransformer: Speech Enhancement Transformer.* Cognitive Computation 14(3), 2022.  
[2] Wang, Z.-Q., et al. *STFT-domain Neural Speech Enhancement with Very Low Algorithmic Latency.* IEEE/ACM TASLP 31, 2022.  
[3] Oruh, J., & Viriri, S. *Spectral Analysis for Automatic Speech Recognition and Enhancement.* Proc. ICMlN, 2020.

