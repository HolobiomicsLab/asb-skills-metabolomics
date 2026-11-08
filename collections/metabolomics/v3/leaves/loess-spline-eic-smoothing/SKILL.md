---
name: loess-spline-eic-smoothing
description: 'Use when deconvolving MS1-only LC/HRMS data with IDSL.CSA: you need
  Pearson correlation coefficients between co-eluting peaks'' EICs to be robust to
  noise and irregular sampling before grouping ions into CSA composite spectra clusters.'
when_to_use_negative:
- Raw unsmoothed chromatograms are already clean and regularly sampled with no need
  for peak grouping into composite spectra
- The analysis targets DDA MS/MS or DIA (MSE, AIF, SWATH) spectra rather than MS1-only
  composite spectra grouping
- Input is an already-annotated peak table where ions are not being grouped by chromatographic
  co-elution correlation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.CSA
  role: R package that implements the LOESS + cubic-spline smoothing and Pearson-correlation
    grouping of co-eluting MS1 EICs into CSA clusters
  repo: https://cran.r-project.org/package=IDSL.CSA
- name: IDSL.IPA
  role: Upstream peak-detection workflow that generates the peaklists and peak_alignment
    directories whose EICs feed the LOESS/spline smoothing step
  repo: https://github.com/idslme/IDSL.IPA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:a32e23cb7e2bf4442913bb23347405471ef8b121370b70578a277c06519e08c9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# loess-spline-eic-smoothing

## Summary

A chromatographic correlation smoothing method in the IDSL.CSA composite-spectra deconvolution pipeline: extracted ion chromatograms (EICs) of co-eluting MS1 peaks are LOESS-smoothed and then cubic-spline interpolated before Pearson correlation, to group correlating ions into CSA clusters.

## When to use

Use when deconvolving MS1-only LC/HRMS data with IDSL.CSA: you need Pearson correlation coefficients between co-eluting peaks' EICs to be robust to noise and irregular sampling before grouping ions into CSA composite spectra clusters.

## When NOT to use

- Raw unsmoothed chromatograms are already clean and regularly sampled with no need for peak grouping into composite spectra
- The analysis targets DDA MS/MS or DIA (MSE, AIF, SWATH) spectra rather than MS1-only composite spectra grouping
- Input is an already-annotated peak table where ions are not being grouped by chromatographic co-elution correlation

## Inputs

- Extracted ion chromatograms (EICs) of MS1 peaks from the IDSL.IPA peaklists and peak_alignment directories
- CSA parameter spreadsheet (CSA0001–CSA0011) selecting parameters including correlation threshold and RT window

## Outputs

- Pearson correlation coefficients between LOESS + cubic-spline smoothed EIC pairs
- CSA clusters of correlating MS1 ions recorded in aligned_spectra_table.csv
- Subsets of aligned peak tables for major ions in each CSA cluster

## How to apply

In the CSA deconvolution workflow, for each seed peak selected by signal-to-noise (S/N) ranking from the IDSL.IPA peak_alignment table, gather candidate co-eluting neighbors within the retention-time window. Extract each candidate's EIC and smooth it with a LOESS regression followed by cubic-spline interpolation to produce continuous, denoised chromatographic traces. Compute Pearson correlations between the smoothed EICs of the seed peak and each neighbor; retain neighbors with correlation ≥ 0.98. Then apply the minimum ion count and m/z range filters (and handle 12C/13C isotopologue m/z spacing) before aggregating qualifying neighbors into CSA clusters written to aligned_spectra_table.csv.

## Related tools

- **IDSL.CSA** (R package that implements the LOESS + cubic-spline smoothing and Pearson-correlation grouping of co-eluting MS1 EICs into CSA clusters) — https://cran.r-project.org/package=IDSL.CSA
- **IDSL.IPA** (Upstream peak-detection workflow that generates the peaklists and peak_alignment directories whose EICs feed the LOESS/spline smoothing step) — https://github.com/idslme/IDSL.IPA

## Examples

```
library(IDSL.CSA); IDSL.CSA_workflow("Address of the CSA parameter spreadsheet")
```

## Evaluation signals

- Correlation values are computed on smoothed (LOESS + cubic-spline interpolated) EICs, not raw chromatograms
- Neighbor retention criterion is Pearson correlation ≥ 0.98 with the seed peak's smoothed EIC
- CSA clusters reproduce the pooled-standards validation counts: 255 peaks, 153 neighbors, 42 clusters
- Output aligned_spectra_table.csv contains the CSA aggregation results on the aligned peak table

## Limitations

- Sensitivity depends on the IDSL.IPA peaklists and peak_alignment directories being correctly generated; missing or noisy EICs degrade correlations
- Applying the 0.98 correlation cutoff assumes chromatographic co-elution implies co-origin; low-intensity or irregular peaks may fail to group
- Isotopologue pairs (12C/13C) require explicit handling of expected m/z spacing or they may be mis-grouped

## Evidence

- [other] The CSA deconvolution mechanism extracts EICs for candidate co-eluting ions and computes Pearson correlations among the grouped peaks' EICs after LOESS smoothing followed by cubic spline interpolation: [evidence span withheld — anchor status: not_found]
- [other] For each seed peak, gather co-eluting candidate neighbors within the retention-time window and compute Pearson correlations between LOESS + cubic-spline smoothed extracted ion chromatograms (EICs), retaining neighbors with correlation >= 0.98: [evidence span withheld — anchor status: not_found]
- [results] Validation: the module output must reproduce the reported 255 peaks, 153 neighbors, and 42 clusters on the pooled-standards test file: [evidence span withheld — anchor status: not_found]
- [readme] Prior to processing your mass spectrometry data (mzXML, mzML, netCDF) using the IDSL.CSA workflow, mass spectrometry data should be processed using the IDSL.IPA workflow to acquire chromatographic information of the peaks (m/z-RT): [evidence span withheld — anchor status: not_found]
