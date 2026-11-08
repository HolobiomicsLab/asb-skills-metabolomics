---
name: linear-model-differential-abundance-analysis
description: Use when a peak-value feature matrix (samples x features) from 1D NMR
  spectra must be tested for features associated with a continuous dose, ordinal diet
  level, or multi-group factor, with multiple-testing correction.
when_to_use_negative:
- Input is already a processed feature table from a different tool whose peak lists
  are intended for MetaboAnalyst or muma-R rather than the speaq 2.0 statistical tools
- The goal is raw spectra alignment or quantitation (speaq <= 1.2.3 CluPA/BW-ratio
  functionality) rather than peak-based differential analysis
- The spectra are severely shifted and only an alignment step (not peak-based modeling)
  is required
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: speaq 2.0
  role: Provides the peak-based workflow (wavelet peak picking, grouping, peak filling)
    whose feature matrix is the input to the linear-model differential analysis with
    Bonferroni correction
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributed implementation of speaq 2.0 whose functions (getWaveletPeaks,
    PeakGrouper, BuildFeatureMatrix) generate the matrix and support differential
    analysis with more than 2 groups
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Runtime environment in which the speaq package functions are called to fit
    the models and export results
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:3776fb662ec320348180056819499e23997ad12975ecae85d3ceabc2043caf41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# linear-model-differential-abundance-analysis

## Summary

Fit a linear model of each peak-value feature against a continuous or multi-group design variable (e.g., increasing onion diet percentage) using the speaq 2.0 feature matrix, and call significant peaks after Bonferroni correction for multiple testing. Applied in the paper to a 31-sample x 677-feature urine 1D NMR matrix, yielding 9 significant peaks queryable against HMDB.

## When to use

Use when a peak-value feature matrix (samples x features) from 1D NMR spectra must be tested for features associated with a continuous dose, ordinal diet level, or multi-group factor, with multiple-testing correction.

## When NOT to use

- Input is already a processed feature table from a different tool whose peak lists are intended for MetaboAnalyst or muma-R rather than the speaq 2.0 statistical tools
- The goal is raw spectra alignment or quantitation (speaq <= 1.2.3 CluPA/BW-ratio functionality) rather than peak-based differential analysis
- The spectra are severely shifted and only an alignment step (not peak-based modeling) is required

## Inputs

- Peak-value feature matrix from speaq 2.0 (e.g. 31 samples x 677 features from 31 urine 1D NMR spectra)
- Design vector (e.g. increasing onion diet percentage per sample, or a class factor for >2 groups)
- ppm vector associated with the features, for HMDB lookup of significant peaks

## Outputs

- Per-feature linear-model statistics with Bonferroni-corrected significance calls (e.g. 9 significant peaks matching Fig 8)
- Feature matrix and differential-analysis results exported to CSV (speaq_onion_results.csv)
- List of ppm values of significant peaks for HMDB biomarker search

## How to apply

First build the feature matrix with the speaq 2.0 workflow: Mexican hat wavelet (CWT-based) peak detection via getWaveletPeaks (no baseline correction or smoothing needed), peak grouping via PeakGrouper, and peak filling via BuildFeatureMatrix, producing e.g. a 31 x 677 peak-value matrix from the onion intake in mice urine spectra. Then fit a linear model of each feature against the increasing onion percentage (the model supports more than 2 groups) and apply Bonferroni correction for multiple testing to declare significant peaks. Optionally run PCA on the same feature matrix to check whether diet groups separate consistently. Write the feature matrix and differential-analysis results (including the significant peaks, e.g. the 9 peaks of Fig 8) to a CSV such as speaq_onion_results.csv, and use the ppm values of significant peaks to query HMDB for candidate biomarkers.

## Related tools

- **speaq 2.0** (Provides the peak-based workflow (wavelet peak picking, grouping, peak filling) whose feature matrix is the input to the linear-model differential analysis with Bonferroni correction) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributed implementation of speaq 2.0 whose functions (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix) generate the matrix and support differential analysis with more than 2 groups) — https://cran.r-project.org/package=speaq
- **R** (Runtime environment in which the speaq package functions are called to fit the models and export results) — https://cran.r-project.org/package=speaq

## Examples

```
library(speaq); Features <- BuildFeatureMatrix(Y.data = PeakGrouper(Y.peaks = getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)))
```

## Evaluation signals

- The number of Bonferroni-significant features equals 9, matching Fig 8 of the paper for the onion intake in mice dataset
- The feature matrix has the expected dimensions (31 samples x 677 features) after peak filling
- The PCA score plot on the feature matrix shows no consistent group separation by diet group, as reported in the paper
- speaq_onion_results.csv contains both the feature matrix and the differential-analysis results including the significant-peak list
- The ppm values of significant peaks are usable as queries against HMDB without further transformation

## Limitations

- The Bonferroni correction is conservative; significance counts depend on the number of features surviving peak filling (677 here)
- Peak grouping can delete large peaks and peak filling may not mitigate all problems; the optional SilhouetR function detects grouping issues via silhouette values per group
- The paper's onion intake example found no consistent PCA separation by diet group, so significant peaks may not correspond to visually separable clusters

## Evidence

- [supplementary] Bonferroni-corrected differential analysis identifying 9 significant peaks: "speaq 2.0 workflow applied to the onion intake in mice data [37]."
- [readme] Feature matrix construction steps: "# peak detection (the default mode is parallel)
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)"
- [readme] Differential analysis capability: [evidence span withheld — anchor status: external_doc]
- [intro] Wavelet rationale avoiding baseline correction and smoothing: [evidence span withheld — anchor status: not_found]
- [intro] HMDB querying of significant peak ppm values: "the ppm values of the actual peaks of interest that can then be used to query a database, like HMDB [24]"
