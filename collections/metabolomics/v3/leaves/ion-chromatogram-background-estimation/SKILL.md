---
name: ion-chromatogram-background-estimation
description: Use when processing LC-MS/MS MRM ion chromatograms where a background
  intensity level is needed to define smoothing windows or section extents prior to
  peak detection and quantitation.
when_to_use_negative:
- The input is not an MRM ion chromatogram but already a peak/intensity table from
  a prior quantitation step.
- A fixed global baseline is required instead of a section-adaptive background, e.g.
  when comparing against tools like TargetLynx that connect nearby valleys.
- The chromatogram data is of a non-MRM acquisition where dwell time is uniform and
  the dynamic dwell-time smoothing rationale does not apply.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Implements the dynamic signal smoothing that consumes the 30th-percentile
    background estimate to size and adjust chromatogram sections
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime environment for MRMQuant; loads binary MAT-files generated from the
    sample data and provides the built-in 'findpeaks' function used after smoothing
- name: ProteoWizard
  role: Converts vendor raw MRM data into mzML/mzXML sample files that MRMQuant loads
    for background estimation and smoothing
- name: msconvert
  role: ProteoWizard command used to convert raw data of various vendors into the
    formats MRMQuant reads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:264a619f86d8f2d98dbcd7df62b5d10ae1a62d25d84dcdf6b54e0f90b8857a60
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MATLAB
  - MRMQuant
  - ProteoWizard
  - msconvert
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ion-chromatogram-background-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates the background intensity of an ion chromatogram as its 30th-percentile signal intensity, a threshold MRMQuant uses for dynamic signal smoothing and section-adaptive adjustment of chromatogram sections.

## When to use

Use when processing LC-MS/MS MRM ion chromatograms where a background intensity level is needed to define smoothing windows or section extents prior to peak detection and quantitation.

## When NOT to use

- The input is not an MRM ion chromatogram but already a peak/intensity table from a prior quantitation step.
- A fixed global baseline is required instead of a section-adaptive background, e.g. when comparing against tools like TargetLynx that connect nearby valleys.
- The chromatogram data is of a non-MRM acquisition where dwell time is uniform and the dynamic dwell-time smoothing rationale does not apply.

## Inputs

- Ion chromatogram (retention time, intensity pairs) from an MRM sample file
- MRM sample file in mzML or mzXML format (converted from vendor raw data via ProteoWizard msconvert)

## Outputs

- Per-section background intensity estimates (30th percentile of signal intensity)
- Section boundaries adjusted so each section starts and ends at signal not higher than 1.2x the background intensity
- Smoothed chromatogram ready for peak detection by MRMQuant's dynamic signal smoothing step

## How to apply

Load the ion chromatogram (retention time, intensity pairs) from an MRM sample file, e.g. one converted from vendor raw data to mzML via ProteoWizard msconvert. Partition the chromatogram into sections and compute each section's background level as its 30th percentile signal intensity. Use the estimated background to drive MRMQuant's dynamic signal smoothing: each chromatogram section is expanded or reduced so it starts and ends at a signal not higher than 1.2 times the background intensity, which adaptively sizes the smoothing windows for MRM data with dynamic dwell time. In MATLAB, the resulting smoothed chromatogram can then feed the built-in 'findpeaks' function to locate possible peak tip locations for the targeted compounds listed in the method file.

## Related tools

- **MRMQuant** (Implements the dynamic signal smoothing that consumes the 30th-percentile background estimate to size and adjust chromatogram sections) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime environment for MRMQuant; loads binary MAT-files generated from the sample data and provides the built-in 'findpeaks' function used after smoothing)
- **ProteoWizard** (Converts vendor raw MRM data into mzML/mzXML sample files that MRMQuant loads for background estimation and smoothing)
- **msconvert** (ProteoWizard command used to convert raw data of various vendors into the formats MRMQuant reads)

## Evaluation signals

- Each chromatogram section's background should equal its 30th-percentile intensity, not the mean or minimum.
- After adjustment, every section should start and end at a signal intensity not higher than 1.2 times its computed background.
- The adjusted section sizes should reflect the section-adaptive rule (expanded when signal is below 1.2x background, reduced when above), consistent with MRMQuant's dynamic signal smoothing.
- The smoothed chromatogram should preserve retention-time alignment so downstream 'findpeaks' can locate peak tips at the expected RTs of the targeted compounds.
- Chromatograms with dynamic dwell time should show adaptively varying section/window sizes rather than a single fixed window.

## Limitations

- The 30th-percentile background estimate is section-adaptive and may behave differently on chromatograms with very few points or heavily coeluted peaks, where MRMQuant's automatic deconvolution is a separate subsequent step.
- Background estimation feeds smoothing but does not itself resolve coeluted peaks; comparison tools that simply connect nearby valleys (e.g. TargetLynx) can over- or under-estimate abundance when coelution occurs.
- The method is defined for MRM data with dynamic dwell time; applying it to uniform-dwell data may not yield the intended section-adaptive behavior.

## Evidence

- [results] the background is determined using the 30th percentile of signal intensity in an ion chromatogram: [evidence span withheld — anchor status: not_found]
- [abstract] In MRMQuant, the MATLAB built-in function: [evidence span withheld — anchor status: not_found]
