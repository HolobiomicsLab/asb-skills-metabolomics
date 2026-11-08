---
name: mrm-chromatogram-reference-sample-alignment
description: Use when preprocessing MRM-mode LC-MS raw data (mzML) whose chromatograms
  drift in retention time between samples, and peak picking or peak scoring requires
  RT-normalized traces before metabolite quantification.
when_to_use_negative:
- Data is not MRM-mode LC-QQQ-MS chromatogram data (e.g. it is already a peak/feature
  table or centroided DDA spectra)
- Chromatograms contain no quantifier trace or lack the at-least-one qualifier transition
  automRm requires for optimal performance
- A brand-new LC method or excessively dirty samples make cross-correlation against
  prototype chromatograms unreliable and manual peak review is needed
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: automRm
  role: Implements the cross-correlation RT-shift calculation and quadratic shift
    fitting inside process_batch(); writes the RT shift table and shiftplots.pdf
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses the mzML files to extract the quantifier/qualifier chromatogram traces
    used for cross-correlation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:fcb0dfc60b422407ddf0d69b10d44b84c71c2ba49d336be9957c4f5d35710c0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - automRm
  - mzR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mrm-chromatogram-reference-sample-alignment

## Summary

Aligns MRM chromatograms across an LC-QQQ-MS batch by calculating per-metabolite RT shifts via cross-correlation against a reference sample, then fitting those shifts as a quadratic function of RT to smooth out implausible per-metabolite deviations.

## When to use

Use when preprocessing MRM-mode LC-MS raw data (mzML) whose chromatograms drift in retention time between samples, and peak picking or peak scoring requires RT-normalized traces before metabolite quantification.

## When NOT to use

- Data is not MRM-mode LC-QQQ-MS chromatogram data (e.g. it is already a peak/feature table or centroided DDA spectra)
- Chromatograms contain no quantifier trace or lack the at-least-one qualifier transition automRm requires for optimal performance
- A brand-new LC method or excessively dirty samples make cross-correlation against prototype chromatograms unreliable and manual peak review is needed

## Inputs

- mzML-derived quantifier and qualifier chromatograms for all samples in an MRM batch (e.g. the HILIC 1a demodata set)
- Expected RTs for each metabolite transition
- Peak classification scores per metabolite per sample (used to select the reference sample)

## Outputs

- Per-sample, per-metabolite raw (cross-correlation) RT shifts
- Quadratic-fitted per-sample, per-metabolite RT shifts
- Optionally, shiftplots.pdf visualizing the RT shift fits

## How to apply

First select the reference sample as the sample with the highest average peak classification score across metabolites (the same selection used in the automRm peak-picking stage). For each metabolite and each sample, calculate the RT shift by simultaneous cross-correlation of both the quantifier and qualifier chromatograms with the corresponding chromatograms of the reference sample within a time window around the expected RT. Then fit the RT shifts of all metabolites in a sample as a quadratic function of RT to eliminate artifacts from improbably pronounced shifts, and repeat the shift calculation with a smaller time window centered on the quadratic-fitted value. This is implemented in automRm's process_batch() workflow and relies on mzR for parsing the mzML files. The rationale is that metabolites eluting close together should shift coherently, so quadratic smoothing removes outliers while remaining faithful to the initial cross-correlation estimates.

## Related tools

- **automRm** (Implements the cross-correlation RT-shift calculation and quadratic shift fitting inside process_batch(); writes the RT shift table and shiftplots.pdf) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses the mzML files to extract the quantifier/qualifier chromatogram traces used for cross-correlation)

## Examples

```
library(automRm); process_batch(get_parameters())
```

## Evaluation signals

- Fitted RT shifts of metabolites eluting close to each other deviate far less than the up-to-30-second deviations observed before fitting
- Quadratic-fitted shift values remain plausible relative to the initial cross-correlation estimates (no extreme departures)
- A table is produced containing both raw and quadratic-fitted per-sample, per-metabolite RT shifts
- The reference sample chosen is the one with the highest average peak classification score across metabolites
- A second cross-correlation pass with a smaller window centered on the fitted value converges (shifts do not move implausibly)

## Limitations

- automRm requires at least one qualifier in addition to the quantifier for optimal performance
- Retreat to manual peak review is needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods
- Quadratic fitting is only justified when shifts vary smoothly across the chromatogram; improbably pronounced per-metabolite shifts are artifacts it is designed to eliminate

## Evidence

- [methods] RT alignment via cross-correlation: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [methods] RT shift fitting as quadratic function: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [methods] Reference sample selection and prototype chromatograms: "sum the RT-shifted chromatograms of a metabolite across all samples"
- [discussion] Qualifier requirement: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [methods] mzR dependency: "automRm depends on several packages such as mzR7 for parsing of mzML files"
