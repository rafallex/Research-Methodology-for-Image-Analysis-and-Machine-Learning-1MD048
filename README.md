# Time-to-Detection for antibiotic susceptibility testing

Coursework for **1MD048 Research Methodology** (period 2, year 1) of the M.Sc. in Image Analysis and Machine Learning at Uppsala University.

The task was to take pre-computed Omnipose segmentation masks of *Mycobacterium smegmatis* growing in a microfluidic chip and work out the earliest time at which a rifampicin-treated population (RIF) can be told apart from an untreated control (REF). That time is the time-to-detection (TTD): the sooner you can call inhibition reliably, the faster a susceptibility test reports.

The masks come from the course (distributed through Studium), so this repo is the analysis code, not the imaging. I started from simple pointwise comparisons and then added a second notebook with methods that model the whole growth curve.

## Notebooks

**`ttd_analysis - first submission version.ipynb`** — the submitted pipeline. Loads the masks, builds per-frame growth time series for REF and RIF (10 min per frame), fits exponential growth rates, and runs four TTD detectors over a grid of thresholds to find the earliest detection:

1. **Relative difference** — flags when (REF − RIF)/REF crosses a threshold
2. **Slope change** — flags divergence in the fitted growth rates
3. **Statistical** — Welch t-test between conditions per frame (p < 0.05)
4. **Ratio threshold** — flags when RIF/REF drops below a threshold

On the susceptible dataset the ratio-threshold method (0.95) gave the earliest stable call at 670 min (≈11.2 h), with REF/RIF growth rates of k = 0.00089 vs 0.00029 min⁻¹ and about 68% growth inhibition.

**`EXP-23-BZ3167_TTD_Implementation_v42.ipynb`** — methods that go beyond pointwise tests, each tied to a reference in the literature:

- Gaussian Process regression on the growth curves (AMiGA-style, Midani 2021 / Tonner 2017)
- PELT changepoint detection (`ruptures`, Truong 2020)
- Bootstrap divergence-point analysis (Efron & Tibshirani 1993)
- Hidden Markov Model state detection (`hmmlearn`, Rabiner 1989)
- Functional data analysis with permutation testing (Ramsay & Silverman 2005)

`ttd_analysis_v40.ipynb` and the `*_fixed_frame30.ipynb` variants are later iterations of the same code; the frame-30 versions rerun the analysis with a corrected starting frame.

## Data

Download the mask data from Studium and extract next to the notebook as `data/`:

```
data/
├── Original_data/      # susceptible strain
│   ├── REF_raw_data.../Pos101/MASK_*.tif
│   └── RIF_raw_data.../Pos101/MASK_*.tif
└── HR/                 # heteroresistant strain
    ├── HR_REF_masks/
    └── HR_RIF10_masks/
```

## Running it

```bash
pip install -r requirements.txt
```

Then open the notebook, set `DATA_ROOT` in the path-configuration cell to your `data/` folder, and run all cells in order. `DATASET_INDEX` switches between the susceptible (0) and HR (1) datasets.

The submission notebook writes its figures and tables to the working directory: `ttd_analysis_results.png`, `ttd_optimization_heatmap.png`, `mask_comparison.png`, and the `analysis_summary.csv` / `growth_timeseries.csv` / `ttd_optimization_results.csv` tables.

The advanced notebook needs `scikit-learn`, `ruptures`, `hmmlearn`, and `seaborn` on top of the base requirements.
