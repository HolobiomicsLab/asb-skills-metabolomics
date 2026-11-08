---
name: explained-variance-reporting
description: Use when a PCA has been (or will be) fit on Protomix-preprocessed urine
  spectra (0.1 ppm rectangular bins, TotalArea-normalized) and you need per-component
  and cumulative explained-variance ratios, e.g. to justify how many PCs to retain
  for downstream cancer-vs-BHP analyses.
when_to_use_negative:
- Input is raw FID or unprocessed spectra — variance ratios on unbinned/unnormalized
  data are not comparable to the ~92.38% (PC1-PC10) reference; run the Protomix chain
  (binning, normalization) first
- The goal is supervised class separation or predictive modeling — explained-variance
  reporting is an unsupervised PCA diagnostic and does not use PC/BHP labels during
  fitting
- You only need spectral preprocessing (peak alignment, baseline correction, binning)
  — that is the Protomix preprocessing skill, not variance reporting
edam_operation: http://edamontology.org/operation_0522
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3855
tools:
- name: scikit-learn
  role: PCA implementation used to fit the model on binned spectra and extract explained_variance_ratio_
- name: Protomix
  role: 'Produces the preprocessed input: 0.1 ppm rectangular binning and normalization
    of 1H-NMR spectra prior to PCA'
  repo: https://github.com/mzniber/protomix
- name: numpy
  role: Numerical handling of the spectra matrix and cumulative variance computation
- name: pandas
  role: Writes the component-wise and cumulative variance table to pca_variance_report.csv
- name: plotly
  role: Interactive visualization of the scree / cumulative explained-variance plot
    in the tutorial workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:59445facf93f2b7516791ddaf632e0d1f9d3e244ca45f4228478fc5e566ed3b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  - plotly
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0522
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3855
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# explained-variance-reporting

## Summary

Quantifies how much total variance in a preprocessed 1H-NMR spectra matrix is captured by the leading principal components, producing a component-wise and cumulative explained-variance table (and scree-style report) after PCA of the binned, normalized spectra.

## When to use

Use when a PCA has been (or will be) fit on Protomix-preprocessed urine spectra (0.1 ppm rectangular bins, TotalArea-normalized) and you need per-component and cumulative explained-variance ratios, e.g. to justify how many PCs to retain for downstream cancer-vs-BHP analyses.

## When NOT to use

- Input is raw FID or unprocessed spectra — variance ratios on unbinned/unnormalized data are not comparable to the ~92.38% (PC1-PC10) reference; run the Protomix chain (binning, normalization) first
- The goal is supervised class separation or predictive modeling — explained-variance reporting is an unsupervised PCA diagnostic and does not use PC/BHP labels during fitting
- You only need spectral preprocessing (peak alignment, baseline correction, binning) — that is the Protomix preprocessing skill, not variance reporting

## Inputs

- Normalized, binned spectra matrix from the Protomix preprocessing chain (0.1 ppm rectangular bins, TotalArea-normalized; samples x binned ppm features)
- Optional: preprocessed_spectra artifact upstream of PCA in the Protomix workflow

## Outputs

- pca_variance_report.csv with per-component explained variance ratio and cumulative explained variance
- Fitted scikit-learn PCA object and scree/cumulative-variance plot (e.g. via plotly)

## How to apply

Load the normalized, binned spectra matrix produced by the Protomix preprocessing chain (0.1 ppm rectangular binning via px.binning, TotalArea normalization) with rows as samples and no class labels used during fitting. Instantiate scikit-learn PCA ('from sklearn.decomposition import PCA') and fit it on the binned spectra unsupervised, then extract explained_variance_ratio_ per component and the cumulative sum. In the prostate-cancer vs benign-prostatic-hyperplasia tutorial data, expect PC1-PC10 to account for ~92.38% of total variance and PC1-PC4 for ~68.25% — use these as reference magnitudes, not exact thresholds, since values shift with binning and normalization choices. Write the component-wise and cumulative variance table to pca_variance_report.csv (optionally visualizing it with plotly, the visualization library used in the Protomix tutorial). Judge correctness by checking that ratios are monotonically non-increasing, sum to 1.0 over all components, and that the leading-component percentages fall in the expected range for this dataset.

## Related tools

- **scikit-learn** (PCA implementation used to fit the model on binned spectra and extract explained_variance_ratio_)
- **Protomix** (Produces the preprocessed input: 0.1 ppm rectangular binning and normalization of 1H-NMR spectra prior to PCA) — https://github.com/mzniber/protomix
- **numpy** (Numerical handling of the spectra matrix and cumulative variance computation)
- **pandas** (Writes the component-wise and cumulative variance table to pca_variance_report.csv)
- **plotly** (Interactive visualization of the scree / cumulative explained-variance plot in the tutorial workflow)

## Examples

```
from sklearn.decomposition import PCA; evr = PCA(n_components=10).fit(bin_df).explained_variance_ratio_; cum = evr.cumsum()
```

## Evaluation signals

- Sum of all explained_variance_ratio_ values equals 1.0 (within floating-point tolerance) and ratios are monotonically non-increasing across components
- For the Protomix tutorial dataset, cumulative variance of PC1-PC10 is ~92.38% and PC1-PC4 is ~68.25% — large deviations indicate a preprocessing mismatch (bin size, normalization)
- pca_variance_report.csv contains one row per fitted component with both per-component and cumulative columns populated
- PCA was fit unsupervised: verify no PC/BHP label column was passed into the feature matrix before fitting
- Reproducing from the same preprocessed_spectra artifact yields identical ratios (deterministic given fixed preprocessing and PCA solver)

## Limitations

- Explained-variance percentages are sensitive to the binning (0.1 ppm rectangular) and normalization (TotalArea) choices upstream; changing these parameters changes the reported ratios
- Variance captured is an unsupervised property of the spectra and does not itself indicate that leading PCs discriminate prostate cancer from BHP samples
- Results are specific to the Zenodo 13467227 NMR dataset and Protomix pipeline; transferring to other NMR platforms or preprocessing chains will yield different values

## Evidence

- [methods] Binning of spectra for statistical analysis: [evidence span withheld — anchor status: not_found]
- [methods] Rectangular 0.1 ppm binning call: "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [readme] Protomix feature list covering binning and normalization: "**Peak Alignment**
- **Binning**
- **Normalization**"
- [abstract] Protomix as NMR preprocessing package: "we have developed Protomix—a Python package designed for metabolomics research"
