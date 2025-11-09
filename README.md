# InSAR Time Series Analysis for Coherent Scatterers (PS & TCS)

This repository contains a Python-based workflow for processing Interferometric Synthetic Aperture Radar (InSAR) time series data. The primary goal is to identify, classify, and analyze different types of coherent scatterers, including stable first-order points (like Permanent Scatterers, PS) and less stable second-order points (like Temporary Coherent Scatterers, TCS).

The workflow uses the Amplitude Dispersion Index (ADI) for initial classification and provides a suite of tools for generating interferograms, filtering phase time series, and interactively visualizing the results. InSAR based TS analysis

## 🌟 Core Features

* **Data Loading:** Loads complex SLC (Single Look Complex) data and corresponding radar geometry from HDF5 files.
* **Scatterer Classification:** Automatically classifies pixels into **first-order** (PS-like) and **TCS** (Temporary Coherent Scatterer) masks based on Amplitude Dispersion Index (ADI) thresholds.
* **Interferogram Generation:** Computes a "star network" of interferograms (IFGs) from the SLC stack, centered on a single master acquisition.
* **Arc Phase Computation:** Calculates the "arc phase" for each pixel, which is the phase time series relative to a stable reference pixel.
* **Phase Filtering:** Includes Savitzky-Golay and Butterworth low-pass filters to smooth noisy phase time series and separate signal from noise.
* **Noise & Coherence Estimation:** Provides functions to estimate temporal coherence and phase noise from the arc phase time series.
* **Interactive Visualization:**
    * **Arc Phase Plotter:** Click any two points (PS or TCS) on the amplitude image to plot their differential phase time series.
    * **Filtered Plotter:** An advanced plotter that compares the raw arc phase with its Butterworth and Savitzky-Golay filtered versions and shows the residuals.
    * **IFG Network Browser:** Click on a point in the temporal/perpendicular baseline plot to instantly view the corresponding interferogram.

## Workflow Pipeline

The main processing pipeline is executed in `arc_phase_plot.py` and relies on helper functions from the other modules.

1.  **Load Data:** The `load_slc_stack` (from `func.py`) function reads the `slcStack.h5` and `geometryRadar.h5` files, extracting the SLC stack, acquisition dates (tbase), perpendicular baselines (pbase), and geometry.
2.  **Compute ADI:** The `compute_adi` function calculates the Amplitude Dispersion Index (ADI) and mean amplitude for every pixel in the stack.
3.  **Select Points:** `select_points` uses the `ADI_THR_PS` and `ADI_THR_TCS` thresholds to create boolean masks (`first_order_mask`, `tcs_mask`) identifying the two classes of scatterers.
4.  **Generate IFG Star Network:** `compute_ifg_coherence_network` creates a set of interferograms, all referenced to a single master image (typically the one in the middle of the time series).
5.  **Select Reference Pixel:** A stable reference pixel is chosen from the `first_order_mask` using `select_reference_pixel`.
6.  **Compute Arc Phases:** `compute_arc_phase` calculates the phase time series for every pixel relative to the chosen reference pixel.
7.  **Filter Phase Time Series:** The noisy `arc_phases` are smoothed using `butter_lowpass_filter` and `sav_golay_smooth` to estimate the underlying signal and reduce noise.
8.  **Visualize (Interactive):** The script can then launch one of the interactive plotters from `Interactive_plot.py` to explore the data.

---

## 📁 File Structure & Module Descriptions

Here is a breakdown of what each file in the repository does.

### `arc_phase_plot.py`
This is the **main executable script** that ties everything together. It defines the main processing parameters (like file paths and ADI thresholds) and runs the step-by-step workflow. Uncomment the plotting sections at the end of the file to launch the interactive visualizations.

### `func.py`
This is the **core processing library** containing most of the key InSAR functions.
* `load_slc_stack`: Loads SLC, dates, baselines, and geometry from HDF5.
* `compute_adi`: Calculates Amplitude Dispersion Index.
* `select_points`: Classifies points based on ADI thresholds.
* `select_reference_pixel`: Finds a suitable reference pixel.
* `compute_ifg_coherence_network`: Creates the IFG star network and coherence map.
* `compute_arc_phase`: Calculates phase relative to the reference.
* `butter_lowpass_filter` & `sav_golay_smooth`: Time series filtering functions.
* `compute_virtual_reference`: Uses a `KDTree` to find nearest TCS neighbors for first-order points (used for more advanced network generation).

### `Interactive_plot.py`
This module contains the classes for **interactive `matplotlib` plots**.
* `InteractiveArcPlot`: Click on the amplitude map to select two points and plot their differential phase time series.
* `InteractiveFilteredArcPlot`: An advanced version that shows four plots: the amplitude map, Savitzky-Golay filtered phase, Butterworth filtered phase, and a comparison of their residuals.
* `InteractiveIFGPlot`: Shows the baseline plot (time vs. perpendicular baseline) and the selected interferogram. Clicking a point on the baseline plot updates the interferogram image.

### `plot.py`
This module contains functions for **static (non-interactive) plotting**.
* `plot_arc_phase`: Plots the arc phase time series for multiple selected points on a grid of subplots.
* `plot_comparison`: Creates plots comparing the original, Butterworth, and Savitzky-Golay filtered phases for selected pixels.
* `plot_point_network`: Visualizes the spatial distribution of first-order (PS) and TCS points and the connections between them.

### `temporal_noise_estimation.py`
This module provides functions for **analyzing the quality** of the phase time series.
* `compute_temporal_coherence`: Estimates coherence for each pixel based on its phase stability over time.
* `estimate_phase_noise`: Estimates the phase noise (standard deviation) for each pixel.
* `compute_adaptive_thresholds`: Uses percentiles to dynamically determine noise and coherence thresholds.
* `filter_noisy_points`: Uses the adaptive thresholds to create filtered masks of reliable PS and TCS points.

### `tests.py`
Contains a standalone function `likelihood_ratio_test` (LRT) for phase-based noise detection, comparing a temporal coherence estimate (`gamma`) with a pre-computed coherence.

---

## 🚀 How to Run

1.  **Install Dependencies:**
    Ensure you have the required Python libraries installed.
    ```bash
    pip install numpy matplotlib scipy h5py networkx cmcrameri
    ```
    *Note: This project also depends on the specialized InSAR libraries `miaplpy` and `spatz`, which may require separate installation.*

2.  **Configure Paths:**
    Open `arc_phase_plot.py` and update the following variables to point to your data:
    * `slc_stack_path`: Path to your `slcStack.h5` file.
    * `geometry_file_path`: This is automatically set to `geometryRadar.h5` in the same directory.

3.  **Set Parameters:**
    In `arc_phase_plot.py`, you can adjust the ADI thresholds:
    * `ADI_THR_PS = 0.4` (Threshold for first-order points)
    * `ADI_THR_TCS = 0.5` (Threshold for TCS points)

4.  **Run the Script:**
    Execute the main script from your terminal:
    ```bash
    python arc_phase_plot.py
    ```

5.  **Enable Interactive Plots:**
    To use the interactive tools, **uncomment** the desired plotting section at the very end of `arc_phase_plot.py` (inside the `if __name__ == "__main__":` block).

    * **For the IFG Network Browser:**
        ```python
        interactiv_plot = InteractiveIFGPlot(ifg_stack, valid_ifg_pairs, tbase_ifg, pbase_ifg, master_idx)
        plt.show()
        ```
    * **For the Filtered Arc Phase Plotter:**
        ```python
        interactive_plot = InteractiveFilteredArcPlot(
            mean_amplitude=mean_amp,          
            arc_phases=arc_phases,             
            butterworth_phases=smoothed_data,    
            savgol_phases=smoothed_data2,             
            first_order_mask=first_order_mask,        
            second_order_mask=tcs_mask # Use tcs_mask       
        )
        plt.show()
        ```
