---
name: spectral-peak-preprocessing-configuration
description: Use when uploading MGF, mzXML, or mzML spectra to run.npanalysis.org
  for a VInSMoC task and you must decide whether to keep or override the advanced
  peak-merging and peak-filtering settings that gate what peaks are scored.
when_to_use_negative:
- Input spectra have already been preprocessed (merged or filtered) before upload
  — re-running the default advanced settings would double-preprocess the peaks
- The task form is being filled only for basic parameters (search mode, ion mode,
  tolerances, charge, minimum score) that do not affect peak lists
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Hosts the advanced peak-merging and peak-filtering settings that configure
    how uploaded mass spectral input is preprocessed before scoring
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Web interface whose task form exposes the Peak Merge Threshold, Minimum Peak
    Count, Peaks Per Window, and Peak Filtering Window parameters
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:fb6ab32493bd9c1bb96a40eefd82d37cb25aafb8a15c4454336033c9ffecb957
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# spectral-peak-preprocessing-configuration

## Summary

Configures VInSMoC's advanced spectral preprocessing — peak merging within the Peak Merge Threshold while preserving Minimum Peak Count peaks, plus peak filtering that retains the Peaks Per Window most intense peaks per Peak Filtering Window — before spectra are scored against candidate molecules. Defaults should be kept unless the user has already preprocessed the uploaded spectra.

## When to use

Use when uploading MGF, mzXML, or mzML spectra to run.npanalysis.org for a VInSMoC task and you must decide whether to keep or override the advanced peak-merging and peak-filtering settings that gate what peaks are scored.

## When NOT to use

- Input spectra have already been preprocessed (merged or filtered) before upload — re-running the default advanced settings would double-preprocess the peaks
- The task form is being filled only for basic parameters (search mode, ion mode, tolerances, charge, minimum score) that do not affect peak lists

## Inputs

- Mass spectrum file in MGF, mzXML, or mzML format uploaded to run.npanalysis.org as a 'Mass Spectrum' data kind
- VInSMoC task form (exact or variable search mode) with advanced settings enabled

## Outputs

- Preprocessed peak list per input scan (merged peaks within Peak Merge Threshold, filtered to Peaks Per Window most intense peaks per Peak Filtering Window) passed to the spectrum–molecule scoring step

## How to apply

In the VInSMoC task form on run.npanalysis.org, expand the advanced settings section, which controls how the mass spectral input is pre-processed before scoring. Peak merging combines peaks that lie within the 'Peak Merge Threshold' m/z distance of one another while guaranteeing at least 'Minimum Peak Count' peaks survive. Peak filtering then keeps only the 'Peaks Per Window' most intense peaks in m/z windows of size 'Peak Filtering Window'. Leave all four parameters at their defaults unless you have already performed preprocessing on your input spectra before uploading, since double-preprocessing can distort the peak lists that VInSMoC scores against candidate molecules.

## Related tools

- **VInSMoC** (Hosts the advanced peak-merging and peak-filtering settings that configure how uploaded mass spectral input is preprocessed before scoring) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Web interface whose task form exposes the Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, and Peak Filtering Window parameters)

## Evaluation signals

- Submitted task reports a minimum score threshold and best-hit results consistent with a spectrum whose peak list was not doubly preprocessed
- All four advanced parameters (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) left at documented defaults unless the user explicitly preprocessed spectra beforehand
- Spectra uploaded as MGF/mzXML/mzML and declared data kind 'Mass Spectrum' parse successfully before the task is launched
- Result page shows the best hit per input scan that exceeded the minimum score threshold, with each molecule linked to its entry in the source chemical database

## Limitations

- Malformed input data can produce an error status during parsing — the input format must match the declared extension
- Larger inputs may spend significant time in the Running state; results pages auto-refresh with exponential backoff until finished
- The advanced settings should not need to be changed often — overriding them without a preprocessing reason can distort scoring

## Evidence

- [other] Default settings also used in benchmark reproduction workflows: [evidence span withheld — anchor status: not_found]
