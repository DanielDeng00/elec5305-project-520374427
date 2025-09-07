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
Remote speech transmission is highly susceptible to various sources of degradation, such as **background noise, packet loss, echo, and channel distortion**. These issues often result in speech signals that are muffled, distorted, or difficult to comprehend.  

Such degradation not only reduces the quality of everyday online communication but also creates serious challenges for critical applications such as **remote education, telemedicine, and collaborative work**. Addressing this problem is therefore both practically relevant and academically significant.  

The goal of this project is to develop a **speech restoration system** using modern deep learning techniques. Specifically, the system will focus on recovering degraded speech signals that have been distorted during long-distance transmission. By employing neural network models such as **convolutional autoencoders** and **Transformer-based architectures**, the project aims to reconstruct intelligible and perceptually natural speech, providing a listening experience that is closer to the original signal.  

---

## 🎯 Background and Motivation  
Traditional speech enhancement and restoration techniques have relied heavily on classical digital signal processing methods, such as **spectral subtraction, Wiener filtering, and statistical noise modeling**. While these methods can effectively reduce simple additive noise, they often fail to handle more complex and nonlinear distortions caused by channel fading, packet loss, and bandwidth constraints.  

Recent advances in **deep learning** have transformed the field of speech enhancement. Neural networks, particularly convolutional and recurrent models, have demonstrated an impressive ability to learn **time–frequency structures** and recover missing or corrupted speech components. More recently, **Transformer-based architectures** have emerged as powerful alternatives for modeling long-term dependencies in audio signals [1].  

The motivation for selecting this topic lies in both **academic value** and **practical impact**:  
- From an academic perspective, the project will explore advanced deep learning techniques in an applied domain, bridging **machine learning, signal processing, and communication systems**.  
- From a practical perspective, the outcome could contribute to more **robust and reliable online communication systems**, with direct applications in **teleconferencing, distance learning, and healthcare**.  

---

## ⚙️ Proposed Methodology  
This project will be implemented entirely in **MATLAB**, which provides a comprehensive set of toolboxes for both signal processing and machine learning. MATLAB will handle the **complete workflow**, including preprocessing, baseline experiments with classical filtering, and the implementation of neural network models.  

1. **Signal Processing**  
   - Apply **Short-Time Fourier Transform (STFT)** [2] to convert time-domain speech signals into spectrograms.  
   - Implement **Wiener filtering** as a baseline reference system.  
   - Compare the baseline with deep learning methods to highlight improvements.  

2. **Machine Learning Models**  
   - **Phase 1**: Implement a **convolutional autoencoder** for mapping degraded to clean spectrograms.  
   - **Phase 2**: Explore **advanced deep architectures** (e.g., Transformer-based) to model long-term dependencies in audio [3].  
   - Evaluate models for perceptual clarity and intelligibility.  

3. **Datasets**  
   - Use **publicly available online datasets** combined with a **self-constructed recorded dataset**.  
   - Simulate **real-world degradations** such as packet loss, additive noise, channel attenuation, and echo.  
   - Ensure evaluation reflects realistic communication scenarios.  

---

## 🎯 Expected Outcomes  
- A **working prototype** that restores degraded speech signals to significantly improved quality.  
- Input: distorted remote speech → Output: enhanced speech.  
- Evaluation metrics:  
  - **Signal-to-Noise Ratio (SNR) improvement**  
  - **Perceptual Evaluation of Speech Quality (PESQ)**  
  - **Short-Time Objective Intelligibility (STOI)**  
- A **public GitHub repository** containing:  
  - Full **source code**  
  - **Audio demonstrations**  
  - **Documentation** for reproducibility and further research  

---

## ⏱️ Timeline (Weeks 5–13)  

| Weeks       | Tasks                                                                 |
|-------------|----------------------------------------------------------------------|
| **5–7**     | Literature review, dataset collection, and baseline Wiener filtering. |
| **8–9**     | Implement convolutional autoencoder, conduct feasibility tests.       |
| **10–11**   | Model optimization, hyperparameter tuning, explore Transformers.      |
| **12–13**   | Final report, GitHub documentation, and audio demo preparation.       |

---

## ✅ Conclusion  
This project addresses a critical challenge in modern communication systems: the **degradation of speech quality during remote transmission**. By combining **traditional signal processing** with **advanced deep learning architectures**, the proposed work aims to restore intelligibility and perceptual quality in degraded speech signals.  

The expected contributions include:  
- A **working prototype**  
- **Objective performance improvements**  
- **Open-source documentation** for the academic and industrial community  

Ultimately, the project has the potential to improve the **robustness and accessibility of remote communication technologies** in an increasingly connected world.  

---

## 📚 References  
[1] Yu, Weiwei, et al. *SETransformer: Speech enhancement transformer.* Cognitive Computation 14.3 (2022): 1152-1158.  
[2] Wang, Zhong-Qiu, et al. *STFT-domain neural speech enhancement with very low algorithmic latency.* IEEE/ACM Transactions on Audio, Speech, and Language Processing 31 (2022): 397-410.  
[3] Oruh, Jane, and Serestina Viriri. *Spectral analysis for automatic speech recognition and enhancement.* International Conference on Machine Learning for Networking. Springer, 2020.  

---
