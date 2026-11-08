---
name: chromatogram-cross-correlation-rt-shift-estimation
description: Use when preprocessing a batch of mzML MRM chromatograms (quantifier
  + qualifier traces) whose peak RTs drift between samples and need alignment before
  peak integration, e.g. HILIC 1a-style demodata batches in automRm.
when_to_use_negative:
- Acquisition lacks at least one qualifier in addition to the quantifier, since automRm's
  cross-correlation requires qualifier traces for optimal performance
- Data are not MRM-mode LC-MS chromatogram batches (e.g. profile MS1 data or single-sample
  files with no reference sample to correlate against)
- Peaks are already aligned/reported and only quality scoring or peak picking is needed
  — use the ML peak-picking/reporting workflow instead
tools:
- name: automRm
  role: Implements the cross-correlation RT shift estimation and the quadratic fit
    of shifts as a function of RT during process_batch() preprocessing
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses the mzML files into the quantifier/qualifier chromatogram traces used
    for cross-correlation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:27853f2044943a9e7992f49871d32effa0ae8b507c95362530ede0442a631484
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - automRm
  - mzR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# chromatogram-cross-correlation-rt-shift-estimation

## Summary

Estimates per-metabolite retention time (RT) shifts in LC-QQQ-MS MRM batches by cross-correlating each sample's quantifier and qualifier chromatograms against a reference sample, then smooths the shifts across the chromatogram by fitting them as a quadratic function of RT. This removes retention-time drift before prototype chromatogram summing and ML-based peak reporting in automRm.

## When to use

Use when preprocessing a batch of mzML MRM chromatograms (quantifier + qualifier traces) whose peak RTs drift between samples and need alignment before peak integration, e.g. HILIC 1a-style demodata batches in automRm.

## When NOT to use

- Acquisition lacks at least one qualifier in addition to the quantifier, since automRm's cross-correlation requires qualifier traces for optimal performance
- Data are not MRM-mode LC-MS chromatogram batches (e.g. profile MS1 data or single-sample files with no reference sample to correlate against)
- Peaks are already aligned/reported and only quality scoring or peak picking is needed — use the ML peak-picking/reporting workflow instead

## Inputs

- mzML files for a batch of LC-QQQ-MS MRM samples, parsed into quantifier and qualifier chromatogram traces (via mzR)
- Metabolism/transition table with expected RT per metabolite (defines the initial cross-correlation search window)
- Per-sample peak classification scores used to select the reference sample

## Outputs

- Per-sample, per-metabolite table of raw (cross-correlation) and quadratic-fitted RT shifts
- RT-shifted chromatograms used for prototype chromatogram summing across samples
- Optional shiftplots.pdf visualizing per-sample RT shifts

## How to apply

Load the mzML-derived quantifier and qualifier chromatograms for all samples (automRm uses mzR for parsing), and select the reference sample as the one with the highest average peak classification score across metabolites. For each metabolite in each sample, calculate the RT shift by simultaneous cross-correlation of the quantifier and qualifier chromatogram with the respective reference-sample chromatograms within a time window around the expected RT. Fit the RT shifts of all metabolites in a sample as a quadratic function of RT to eliminate artifacts from improbably pronounced per-metabolite shifts. Then repeat the shift calculation with a smaller time window centered on the quadratic-fitted value, and write a table of raw and fitted per-sample, per-metabolite shifts (optionally rendering shiftplots.pdf). The quadratic smoothing rationale is that metabolites eluting close to each other should have similar shifts, so raw cross-correlation estimates deviating by up to ~30 s are treated as artifacts.

## Related tools

- **automRm** (Implements the cross-correlation RT shift estimation and the quadratic fit of shifts as a function of RT during process_batch() preprocessing) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses the mzML files into the quantifier/qualifier chromatogram traces used for cross-correlation)

## Examples

```
library(automRm); process_batch()  # preprocesses the mzML MRM batch, including cross-correlation RT shift estimation and quadratic fitting
```

## Evaluation signals

- Fitted RT shifts of metabolites eluting close to each other deviate far less than the up-to-30-second deviations observed before quadratic fitting
- Fitted shift values remain plausible relative to the initial cross-correlation estimates (no outliers from improbably pronounced per-metabolite shifts)
- The second-pass shift calculation, using a smaller window centered on the fitted value, converges near the fitted shifts rather than diverging
- The output table contains both raw and quadratic-fitted per-sample, per-metabolite RT shifts, and shiftplots.pdf (if rendered) shows smooth shift curves across the chromatogram

## Limitations

- Raw cross-correlation shifts can deviate by up to ~30 s before fitting, so skipping the quadratic smoothing step leaves artifacts from improbably pronounced shifts
- automRm requires at least one qualifier in addition to the quantifier for optimal RT alignment and peak scoring performance
- For excessively dirty samples, very-low-input samples, or new LC methods, RT alignment/peak picking may be unreliable and manual peak review may still be needed

## Evidence

- [methods] calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [methods] RT shifts of all metabolites in a sample are fitted as a quadratic function of RT: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [methods] automRm depends on several packages such as mzR7 for parsing of mzML files: "automRm depends on several packages such as mzR7 for parsing of mzML files"
- [abstract] fully automatic preprocessing of LC-MS data recorded in MRM mode: "fully automatic preprocessing of LC-MS data recorded in MRM mode"
