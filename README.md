# GW150914 Gravitational Wave Analysis Pipeline

This project is an end-to-end analysis of the famous **GW150914** gravitational-wave event, the first direct detection of gravitational waves by LIGO. The notebook builds a complete educational pipeline that starts from raw strain data and ends with:

- a visible chirp in the time-frequency plane,
- an estimate of the **chirp mass** from the observed frequency evolution,
- an uncertainty estimate using bootstrap resampling,
- and a matched-filter confirmation using a compact-binary waveform template.

The goal of this project is to show, step by step, how a gravitational-wave signal can be extracted from noisy detector data and connected back to the physics of a binary black hole merger.

---

## Overview

Gravitational waves are extremely weak distortions in spacetime, so the raw detector data is dominated by noise. In this notebook, the data is processed through a sequence of signal-processing and physics-based steps:

1. **Fetch raw LIGO strain data** from GWOSC for Hanford (H1) and Livingston (L1).
2. **Whiten** the strain to flatten the detector noise spectrum.
3. **Bandpass filter** the signal to keep the useful frequency range.
4. **Compute Q-transform spectrograms** to make the chirp visible.
5. **Estimate the detector PSD** from off-source data using Welch’s method.
6. **Extract a clean frequency track** from the chirp in the spectrogram.
7. **Estimate the frequency derivative** \(df/dt\).
8. **Infer the chirp mass** using the leading-order post-Newtonian formula.
9. **Bootstrap the estimate** to get a rough uncertainty interval.
10. **Perform matched filtering** with a template waveform to verify the signal.

This workflow is not full Bayesian parameter estimation, but it is a strong educational demonstration of the main ideas used in gravitational-wave astronomy.

---

## Event Analyzed

The notebook focuses on:

- **GW150914**
- Event time: **GPS 1126259462.4**
- Detectors used: **H1** and **L1**

This event corresponds to the merger of two black holes and produces the characteristic rising **chirp** in frequency and amplitude.

---

## What the Notebook Does

The notebook is organized into the following major stages:

### 1. Data acquisition
The strain time series is downloaded using `gwpy` from public GWOSC data.

### 2. Signal conditioning
The raw data is whitened and bandpassed to suppress noise outside the relevant frequency region.

### 3. Time-frequency visualization
A Q-transform is used to reveal the chirp as a bright track in the time-frequency plane.

### 4. Chirp-track extraction
A custom algorithm scans the spectrogram and extracts a frequency-vs-time track using:
- an SNR threshold,
- a frequency search window,
- and a continuity constraint.

### 5. Mass estimation
The extracted track is used to estimate \(df/dt\), which is inverted to obtain the chirp mass.

### 6. Uncertainty estimation
A bootstrap procedure perturbs the measured track points by the spectrogram resolution to obtain a 90% confidence interval.

### 7. Matched-filter verification
A PyCBC template waveform is matched against H1 data to confirm that the signal is consistent with a compact binary coalescence.

---

## Main Functions in the Pipeline

## `fetch_strain(detector, t_center, half_window, cache_dir=CACHE_DIR)`

Downloads strain data for a given detector and time window.  
If the data has already been downloaded, it loads the cached file instead.

### Purpose
This is the data ingestion layer of the notebook.

### Why it matters
It makes the analysis reproducible and avoids downloading the same strain repeatedly.

---

## `whiten(strain)`

Whitens the strain time series by dividing by the noise amplitude spectral density.

### Purpose
To flatten the noise spectrum and make the GW signal easier to see.

### Why it matters
Raw LIGO noise is strongly frequency-dependent. Whitening reduces that imbalance.

---

## `bandpass(strain, f_low=30.0, f_high=400.0)`

Applies a bandpass filter to retain only the useful frequency range.

### Purpose
To remove low-frequency and high-frequency noise that is not helpful for this event.

### Why it matters
GW150914 is most visible in the rough 30–400 Hz range.

---

## `estimate_psd(detector, t_end_event, segment_duration=512, cache_dir=CACHE_DIR)`

Estimates the detector’s noise power spectral density using off-source data and Welch’s method.

### Purpose
The PSD is required for matched filtering, since it describes how noise power is distributed across frequency.

### Why it matters
Matched filtering is only optimal when the noise statistics are properly modeled.

---

## `extract_frequency_track(qtransform, t_center, half_window=0.25, snr_threshold=15.0, f_min_search=55.0, continuity_tol=15.0)`

Extracts a clean chirp track from the Q-transform spectrogram.

### Purpose
To identify the bright frequency ridge corresponding to the gravitational-wave chirp.

### How it works
For each time slice:
- it searches for the maximum power frequency,
- rejects weak slices using an SNR threshold,
- restricts the search to a frequency window,
- and enforces smooth continuity from one accepted point to the next.

### Why it matters
This converts a visual spectrogram feature into quantitative \((t, f)\) data.

---

## `pick_point(t_arr, f_arr, t_target, window=0.005)`

Selects the point closest to a target time within a small window.

### Purpose
To choose stable points from the extracted track for slope estimation.

### Why it matters
The chirp-mass estimate uses two representative points on the inspiral track.

---

## `compute_dfdt(f1, f2, t1, t2)`

Computes a finite-difference estimate of the frequency derivative.

### Purpose
To estimate how quickly the GW frequency is increasing.

### Why it matters
The chirp mass is directly related to \(df/dt\) through post-Newtonian inspiral theory.

---

## `chirp_mass_from_dfdt(dfdt, f_ref)`

Inverts the leading-order post-Newtonian formula to estimate the chirp mass.

### Purpose
To convert the measured inspiral rate into a physical source parameter.

### Why it matters
The chirp mass is the best-measured mass combination in the inspiral phase.

---

## Data Processing Steps in Detail

### Raw strain
The notebook begins by fetching raw strain for both LIGO detectors around the event time.

### Whitening
The raw strain is whitened so the spectral noise is flattened.

### Bandpass filtering
The whitened strain is filtered between 30 and 400 Hz.

### Q-transform
A Q-transform spectrogram is computed for both detectors to reveal the chirp track.

### PSD estimation
A PSD is estimated from 512 seconds of off-source data ending 1 second before the merger.

### Frequency-track extraction
The H1 spectrogram is scanned and a clean frequency track is extracted using the custom gating rules.

### Two-point inspiral estimate
Two reference points are selected from the track and used to estimate \(df/dt\).

### Chirp mass estimate
The chirp mass is calculated from the leading-order inspiral formula.

### Bootstrap uncertainty
The estimate is resampled 10,000 times using the spectrogram’s bin resolution to obtain a 90% confidence interval.

### Matched filtering
A SEOBNRv4 template is matched against H1 data to verify that the signal is consistent with a compact binary merger.

---

## Physical Interpretation

The signal shows the classic gravitational-wave chirp:

- the frequency rises with time,
- the amplitude grows as the black holes spiral closer,
- and the track ends in a merger/ringdown phase.

This is the signature of a **binary black hole coalescence**.

The chirp mass estimate is based on the leading-order relation

\[
\frac{df}{dt} = \frac{96}{5}\pi^{8/3}\left(\frac{G\mathcal{M}}{c^3}\right)^{5/3} f^{11/3}
\]

which is inverted in the notebook to solve for \(\mathcal{M}\).

---

## Figures Produced by the Notebook

The notebook saves the following figures:

- `raw_strain.png` — raw strain for both detectors
- `whitened_strain.png` — whitened and bandpassed strain
- `qtransform.png` — spectrograms showing the chirp
- `psd.png` — detector noise ASD estimated with Welch’s method
- `freq_track.png` — extracted chirp track with 0PN overlay
- `chirp_mass.png` — chirp mass comparison with published value
- `bootstrap.png` — bootstrap distribution of chirp mass
- `matched_filter_snr.png` — matched-filter SNR time series

---

## Example Results

The notebook is designed to produce results close to the published GW150914 values:

- **Chirp mass** close to the published value of **28.3 \(M_\odot\)**
- **Matched-filter peak SNR** showing a strong event near merger time
- A chirp track that follows the theoretical inspiral trend

Exact values depend on the extracted track points and the resolution of the Q-transform.

---

## Dependencies

The notebook uses:

- `numpy`
- `matplotlib`
- `scipy`
- `gwpy`
- `gwosc`
- `pycbc`

The notebook also installs `gwpy`, `gwosc`, and `pycbc` inside the first cells if they are not already available.

---

## How to Run

1. Open the notebook in Jupyter or Colab-compatible environment.
2. Run all cells in order.
3. Make sure internet access is available for the first data download from GWOSC.
4. The notebook will generate all figures and save them as PNG files.

---

## Project Summary

This project demonstrates a full gravitational-wave analysis workflow on GW150914. It shows how to:

- fetch real detector data,
- reduce noise,
- reveal a chirp in time-frequency space,
- extract a physical frequency track,
- estimate source properties from inspiral dynamics,
- and verify the result using matched filtering.

The notebook is a compact but meaningful demonstration of how gravitational-wave astronomy works in practice.

---

## Acknowledgements

- **LIGO Scientific Collaboration**
- **GWOSC** for open public strain data
- **GWPy** and **PyCBC** for gravitational-wave analysis tools
- Abbott et al. (2016) for the GW150914 discovery paper
