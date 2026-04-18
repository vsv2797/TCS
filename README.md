# TCS: Temporary Coherent Scatterers in InSAR Time Series

This repository provides a comprehensive framework for identifying, classifying, and processing **Temporary Coherent Scatterers (TCS)** alongside traditional **Permanent Scatterers (PS)**. It serves as both a processing toolkit and a conceptual guide for modern Persistent Scatterer Interferometry (PSI).

---

## 📘 Understanding TCS vs. PS

In traditional PSI, the focus is on pixels that remain stable over the entire observation period (PS). However, many environments contain "temporary" stability (TCS), which are points that are coherent only for a portion of the time series.

| Feature | Permanent Scatterers (PS) | Temporary Coherent Scatterers (TCS) |
| :--- | :--- | :--- |
| **Stability** | Consistent over the entire time series. | High coherence for only a sub-period of time. |
| **Typical Objects** | Buildings, bridges, metallic structures. | Construction sites, moving stockpiles, seasonal vegetation. |
| **ADI Range** | Low ($D_A < 0.4$). | Moderate ($0.4 \leq D_A < 0.5$). |
| **Network Role** | The "backbone" of the deformation network. | Increases spatial density in rapidly changing areas. |

---

## 🏗️ The Processing Pipeline

The code in this repository implements a specific workflow to integrate TCS into InSAR analysis:

1.  **Classification via ADI**: Uses the Amplitude Dispersion Index ($D_A = \sigma_A / \mu_A$) to separate points into `first_order_mask` (PS) and `tcs_mask` (TCS).
2.  **Virtual Reference Strategy**: Because TCS may not be stable relative to a global master for the whole period, they are linked to nearby stable PS "anchors" using a $K$-Nearest Neighbor (KNN) approach.
3.  **Arc Phase Computation**: Calculates the phase difference between the TCS and its linked PS to isolate local deformation.
4.  **Temporal Filtering**: Employs Savitzky-Golay and Butterworth low-pass filters to separate the underlying deformation signal from high-frequency noise.

---

## ⚠️ Common Problems & Challenges

Processing TCS introduces several hurdles not found in standard PS processing:

* **Phase Discontinuities**: Because TCS appear or disappear, their phase history is "broken," making temporal phase unwrapping difficult.
* **Lower Coherence**: TCS are inherently noisier; without strict temporal filtering, they can introduce "pseudo-deformation" into the results.
* **Reference Linking**: A TCS is only as good as the PS it is linked to. If the nearest PS is too far away, atmospheric residuals will dominate the arc phase.
* **Signal Masking**: It is challenging to distinguish between a pixel becoming incoherent due to surface change (true TCS behavior) and temporary environmental factors like snow.

---

## 🛠️ Repository Features

### Advanced Filtering & Smoothing (`func.py`)
* **Butterworth Low-pass**: Best for removing high-frequency noise while keeping long-term trends.
* **Savitzky-Golay**: Superior for preserving local signal shapes, such as sudden subsidence steps.

### Interactive Visualization (`Interactive_plot.py`)
* **Filtered Arc Plotter**: Interactively compare raw arc phases against filtered versions and view residuals.
* **IFG Network Browser**: Visualize the star network geometry and inspect individual interferogram quality.

### Noise Estimation (`temporal_noise_estimation.py`)
* **Temporal Coherence**: Calculates $\gamma$ to assess the reliability of points.
* **Adaptive Thresholding**: Dynamically determines noise and coherence cut-offs based on the data's statistical distribution.

---

## 🚀 Getting Started

### Installation
```bash
pip install numpy matplotlib scipy h5py networkx cmcrameri
