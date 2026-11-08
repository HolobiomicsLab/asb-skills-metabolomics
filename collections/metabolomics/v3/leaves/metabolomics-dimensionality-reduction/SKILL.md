---
name: metabolomics-dimensionality-reduction
description: Use when you have a normalized, binned NMR spectra matrix (0.1 ppm rectangular
  bins, TotalArea-normalized) and need an unsupervised variance decomposition or a
  scree/cumulative-explained-variance summary before downstream classification.
when_to_use_negative:
- Input is raw Bruker FIDs or unprocessed spectra that still require the Protomix
  preprocessing chain (group delay removal, apodization, Fourier transform, baseline
  correction) before PCA
- The goal is supervised class discrimination between PCa and BPH groups — PCA fits
  without labels and cannot provide class predictions
- The data are already reduced to a low-dimensional feature table rather than a binned
  spectra matrix
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: PCA implementation used to fit the binned spectra and extract explained_variance_ratio_
- name: Protomix
  role: Produces the binned, normalized spectra matrix (px.binning with bin_size=0.1,
    rectangular) that is the PCA input
  repo: https://github.com/mzniber/protomix
- name: numpy
  role: Numerical support for cumulative variance computation
- name: pandas
  role: Holds the binned spectra DataFrame and writes the pca_variance_report.csv
    table
- name: plotly
  role: Interactive visualization of PCA results such as score plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:39abd53ed5cb65051aa7edbccb5c5b74d74c663b0798301f9dce816c165b91c1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metabolomics-dimensionality-reduction

## Summary

Unsupervised PCA decomposition of Protomix-preprocessed 1H-NMR urine spectra (prostate cancer vs benign prostatic hyperplasia) to quantify how much total variance is captured by the leading principal components, e.g. ~92.38% for PC1-PC10 and ~68.25% for PC1-PC4.

## When to use

Use when you have a normalized, binned NMR spectra matrix (0.1 ppm rectangular bins, TotalArea-normalized) and need an unsupervised variance decomposition or a scree/cumulative-explained-variance summary before downstream classification.

## When NOT to use

- Input is raw Bruker FIDs or unprocessed spectra that still require the Protomix preprocessing chain (group delay removal, apodization, Fourier transform, baseline correction) before PCA
- The goal is supervised class discrimination between PCa and BPH groups — PCA fits without labels and cannot provide class predictions
- The data are already reduced to a low-dimensional feature table rather than a binned spectra matrix

## Inputs

- Preprocessed spectra matrix from the Protomix chain (samples x 0.1 ppm rectangular-binned, TotalArea-normalized intensities), e.g. preprocessed_spectra artifact
- Optional sample metadata (prostate cancer vs benign prostatic hyperplasia labels) for post-hoc interpretation only

## Outputs

- Per-component and cumulative explained-variance table written to pca_variance_report.csv
- PCA scores and loadings for inspection of sample clustering and contributing spectral bins

## How to apply

Load the preprocessed_spectra artifact produced by the Protomix chain — after binning with px.binning(bin_size=0.1, method='rectangular') and TotalArea normalization — as a samples x binned-ppm DataFrame. Instantiate scikit-learn PCA ('from sklearn.decomposition import PCA') and fit it on the binned spectra without using the PCa/BPH labels, since PCA is an unsupervised decomposition. Extract per-component explained_variance_ratio_ and cumulative sums for the leading components. Validate against the reference result: the first 10 principal components should account for ~92.38% of total variance and PC1-PC4 for ~68.25%; write the component-wise and cumulative table to pca_variance_report.csv. Large explained variance concentrated in few components indicates the preprocessing (binning + normalization) produced a compact, analysis-ready feature space.

## Related tools

- **scikit-learn** (PCA implementation used to fit the binned spectra and extract explained_variance_ratio_)
- **Protomix** (Produces the binned, normalized spectra matrix (px.binning with bin_size=0.1, rectangular) that is the PCA input) — https://github.com/mzniber/protomix
- **numpy** (Numerical support for cumulative variance computation)
- **pandas** (Holds the binned spectra DataFrame and writes the pca_variance_report.csv table)
- **plotly** (Interactive visualization of PCA results such as score plots)

## Examples

```
from sklearn.decomposition import PCA; pca = PCA(); pca.fit(binned_spectra_df); print(pca.explained_variance_ratio_.cumsum()[:10])
```

## Evaluation signals

- Cumulative explained variance of PC1-PC10 is approximately 92.38% of total variance
- Cumulative explained variance of PC1-PC4 is approximately 68.25%
- Per-component explained_variance_ratio_ values are non-negative and sum to 1 across all components
- PCA fit uses only the binned spectra rows; PCa/BPH labels are never passed to the fitter
- pca_variance_report.csv contains one row per retained component with both individual and cumulative ratios

## Limitations

- PCA is unsupervised: high explained variance does not imply separation of prostate cancer from benign prostatic hyperplasia samples
- Results depend on upstream preprocessing choices (0.1 ppm rectangular binning, TotalArea normalization); different bin sizes or normalization change the variance profile
- The reference values (92.38% / 68.25%) come from this specific Zenodo NMRDataset (record 13467227) and tutorial workflow; other datasets will differ

## Evidence

- [methods] PCA fit on binned spectra: "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [other] Reference explained-variance result: [evidence span withheld — anchor status: not_found]
- [other] scikit-learn PCA import from tutorial: [evidence span withheld — anchor status: not_found]
- [readme] Protomix scope for NMR preprocessing: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
