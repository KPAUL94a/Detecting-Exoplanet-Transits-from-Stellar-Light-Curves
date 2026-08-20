# Detecting Exoplanet Transits from Stellar Light Curves
 
*by team CYGNUS X*
 
A signal-processing pipeline that scans synthetic stellar light curves for the periodic brightness dips characteristic of a transiting exoplanet, and flags each star as a transit **candidate** or **non-candidate**.
 
## Overview
 
When a planet passes in front of its host star (a *transit*), the star's observed brightness (flux) dips slightly and periodically. This project implements a classical (non-ML) detection pipeline that:
 
1. Reads raw flux time series for 500 stars,
2. Removes the slow-varying stellar baseline (detrending),
3. Smooths out high-frequency noise,
4. Flags statistically significant brightness dips, and
5. Classifies each star as a transit candidate based on how many qualifying dips it has.
The full workflow, from raw data to final classification, lives in a single Jupyter notebook: [`astc_1.ipynb`](astc_1.ipynb).
 
## How It Works
 
| Step | What happens |
|---|---|
| **1.Load data**  | `binary_exoplanet_dataset.csv` is read with pandas; each row is one star's flux time series. |
| **2.Reshape**    | Each row is converted into its own `time` vs. `flux` DataFrame (time is the sample index, since the dataset has no real timestamps). |
| **3.Detrend**    | A rolling median (window = 80 samples) estimates the star's slow-varying baseline brightness, which is subtracted off to flatten the curve. |
| **4.Smooth**     | The detrended flux is smoothed with a **Savitzky–Golay filter** (window = 15, polyorder = 2), compared against a simple moving average. |
| **5.Threshold**  | A per-star dip threshold is set at `median − 3σ` of the smoothed flux. |
| **6.Detect dips**| `scipy.signal.find_peaks` is run on the *inverted* flux to locate dips that fall below the threshold, are spaced at least 50 samples apart, and have sufficient prominence (≈1.5σ). |
| **7.Classify**   | A star is labeled a **Candidate** if it has 2 or more qualifying dips, otherwise **Non-Candidate**. Results for all 500 stars are written to `result.csv`. |
 
On the provided dataset, this pipeline flags **252 of 500** stars as transit candidates.
 
## Repository Structure
 
```
.
├── astc_1.ipynb                  # Main analysis notebook (data load → detection → results)
├── binary_exoplanet_dataset.csv  # Input dataset: 500 stars x 1000 flux samples each
├── result.csv                    # Output: per-star classification results
├── lightcurves_plot/             # Raw light curve plots (flux vs. time) per star
├── curvesmoothing/               # Moving-average vs. Savitzky-Golay smoothing comparison plots
└── dip_detection_result/         # Plots showing detected dips against the dynamic threshold
```
 
## Dataset
 
`binary_exoplanet_dataset.csv` contains:
 
- `case_id` — unique identifier for each star (0–499)
- `t_0` … `t_999` — 1,000 normalized flux measurements per star, forming that star's light curve
## Output
 
`result.csv` contains one row per star:
 
| Column | Description |
|---|---|
| `case_id` | Star identifier, matches the input dataset |
| `is_candidate` | `1` if flagged as a transit candidate, else `0` |
| `label` | `"Candidate"` or `"Non-Candidate"` |
| `dip_count` | Number of qualifying dips detected |
| `dip_indices` | Time-sample indices where dips were detected |
 
## Key Parameters
 
These are the tunable constants used in the detection pipeline (set inside the notebook):
 
| Parameter | Value | Purpose |
|---|---|---|
| Detrending window | 80 samples | Rolling-median window for estimating the stellar baseline |
| Savitzky–Golay window / polyorder | 15 / 2 | Noise smoothing after detrending |
| Dip threshold | median − 3σ | Flux level below which a sample is considered a possible dip |
| Minimum peak distance | 50 samples | Prevents counting the same dip twice |
| Minimum peak prominence | ~1.5σ | Ensures a dip stands out from local noise |
| Minimum dips for "Candidate" | 2 | A star needs at least 2 qualifying dips to be flagged |
 
## Getting Started
 
### Requirements
 
- Python 3
- Jupyter Notebook / JupyterLab
- `numpy`
- `pandas`
- `matplotlib`
- `scipy`
Install dependencies:
 
```bash
pip install numpy pandas matplotlib scipy jupyter
```
 
### Running the Notebook
 
```bash
git clone https://github.com/KPAUL94a/Detecting-Exoplanet-Transits-from-Stellar-Light-Curves.git
cd Detecting-Exoplanet-Transits-from-Stellar-Light-Curves
jupyter notebook astc_1.ipynb
```
 
Run all cells in order. The notebook reads `binary_exoplanet_dataset.csv`, walks through detrending, smoothing, and dip detection (with plots along the way), and writes the final classifications to `result.csv`.

## Credits
 
Built by **team CYGNUS X**.
