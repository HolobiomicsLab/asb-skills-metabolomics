---
name: quadratic-regression-fitting-of-retention-time-shifts
description: Use when per-metabolite RT shifts obtained by cross-correlation of quantifier/qualifier
  MRM chromatograms against a reference sample show implausibly pronounced, non-smooth
  deviations across the chromatogram and must be regularized before peak picking or
  reporting.
when_to_use_negative:
- Data are not MRM-mode LC-MS chromatograms (e.g. DDA/DIA or GC-MS runs without quantifier/qualifier
  traces), since the fitting relies on per-metabolite cross-correlation of quantifier
  and qualifier chromatograms
- The batch has only a quantifier and no qualifier transitions, since automRm requires
  at least one qualifier for optimal performance
- You need an absolute RT standardization across heterogeneous methods rather than
  smoothing sample-specific shifts within one batch
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: automRm
  role: Implements the cross-correlation RT shift calculation and quadratic fitting
    of RT shifts as part of process_batch()
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses mzML files to provide the quantifier/qualifier chromatograms on which
    RT shifts are computed
- name: R
  role: Runtime environment (tested with R 3.6.0 and 4.0.2) in which the fitting is
    executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:90bce33cb21724bced7f28ee53286944e738bab40f49f8e31d347d00364a37b9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - mzR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# quadratic-regression-fitting-of-retention-time-shifts

## Summary

Smooths per-metabolite cross-correlation retention time shifts across an LC-MS MRM chromatogram by fitting the RT shifts of all metabolites in a sample as a quadratic function of RT, eliminating artifacts from improbably pronounced individual shifts before peak picking/reporting in automRm.

## When to use

Use when per-metabolite RT shifts obtained by cross-correlation of quantifier/qualifier MRM chromatograms against a reference sample show implausibly pronounced, non-smooth deviations across the chromatogram and must be regularized before peak picking or reporting.

## When NOT to use

- Data are not MRM-mode LC-MS chromatograms (e.g. DDA/DIA or GC-MS runs without quantifier/qualifier traces), since the fitting relies on per-metabolite cross-correlation of quantifier and qualifier chromatograms
- The batch has only a quantifier and no qualifier transitions, since automRm requires at least one qualifier for optimal performance
- You need an absolute RT standardization across heterogeneous methods rather than smoothing sample-specific shifts within one batch

## Inputs

- mzML-derived quantifier and qualifier chromatograms for all samples in an MRM batch (e.g. the HILIC 1a demodata set)
- Per-metabolite, per-sample raw RT shifts from cross-correlation against the reference sample
- Expected RT of each metabolite (MRM transition table)

## Outputs

- Table of raw and quadratic-fitted per-sample, per-metabolite RT shifts
- Quadratic-fitted RT shift value used as the center for the second, narrower cross-correlation window
- Optional shiftplots.pdf diagnostic plots

## How to apply

First compute raw RT shifts separately for each metabolite in each sample by simultaneous cross-correlation of the quantifier and qualifier chromatograms (parsed from mzML via mzR) with the corresponding chromatograms of the reference sample — the sample with the highest average peak classification score — within a time window around the expected RT. Then fit all metabolite RT shifts within a sample as a quadratic function of RT; metabolites eluting near each other should therefore end up with very similar fitted shifts. Re-run the cross-correlation with a smaller time window centered on the quadratic-fitted value to refine the alignment. Finally write a table of raw and quadratic-fitted per-sample, per-metabolite RT shifts and optionally render shiftplots.pdf for inspection.

## Related tools

- **automRm** (Implements the cross-correlation RT shift calculation and quadratic fitting of RT shifts as part of process_batch()) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses mzML files to provide the quantifier/qualifier chromatograms on which RT shifts are computed)
- **R** (Runtime environment (tested with R 3.6.0 and 4.0.2) in which the fitting is executed)

## Examples

```
Rscript -e "automRm::process_batch(get_ms_type='qqq', printplots=TRUE)"
```

## Evaluation signals

- Fitted RT shifts of metabolites eluting close to each other deviate by far less than the up-to-30-second deviations observed before fitting
- Fitted shift values remain plausible relative to the initial cross-correlation estimates
- The second cross-correlation pass with the smaller window centered on the fitted value converges (shift estimates stable)
- A table with raw and quadratic-fitted per-sample, per-metabolite shifts is produced, and shiftplots.pdf (if rendered) shows smooth shift curves per sample

## Limitations

- Per-metabolite cross-correlation shifts can deviate by up to ~30 seconds before fitting, so the quadratic model must be trusted to suppress these artifacts — a metabolite with a genuinely extreme shift may be pulled toward the fitted curve
- For optimal performance automRm requires at least one qualifier in addition to the quantifier; without qualifiers the shift estimation that feeds the fitting is degraded
- For excessively dirty samples, very-low-input samples, or new LC methods, retreat to manual peak review may still be needed despite the smoothed alignment

## Evidence

- [methods] quadratic fitting of RT shifts: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [methods] cross-correlation RT shift source: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [abstract] package scope: "Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode."
- [methods] mzML parsing dependency: "automRm depends on several packages such as mzR7 for parsing of mzML files"
