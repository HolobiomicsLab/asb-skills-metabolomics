---
name: section-adaptive-signal-smoothing
description: Use when smoothing MRM ion chromatograms that contain dynamic dwell time
  and varying signal/noise levels, where a fixed-size smoothing window would either
  leave noise in low-signal sections or clip true peaks in high-signal sections.
when_to_use_negative:
- Input is not raw MRM ion chromatogram signal but already a peak table or quantitation
  result — smoothing is not meaningful there.
- The chromatogram has uniform dwell time and homogeneous noise so a fixed-window
  smoother suffices and the section-adaptive background rule adds no value.
- You are quantitating coeluted or saturated peaks — MRMQuant's separate deconvolution
  and regression steps handle those cases, not the smoothing step.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MRMQuant
  role: Implements dynamic signal smoothing over MRM ion chromatograms with dynamic
    dwell time, using the section-adaptive background rule before peak detection
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime environment in which MRMQuant's smoothing is executed; chromatogram
    data are held in a binary MATLAB MAT-file
- name: ProteoWizard (msconvert)
  role: Converts vendor MRM raw data into mzML or MAT-file-compatible formats so the
    ion chromatograms can be fed into MRMQuant's smoothing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:6ea9a5a84ba4fe9641932de7fc763e0cb6a841bfc2c7d01927605ea914a8012b
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
  - ProteoWizard (msconvert)
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Section-adaptive signal smoothing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions an MRM ion chromatogram into sections, estimates each section's background as its 30th-percentile intensity, and adapts the smoothing window so each section starts and ends at a signal not higher than 1.2 times its estimated background — the dynamic smoothing step used by MRMQuant before peak detection.

## When to use

Use when smoothing MRM ion chromatograms that contain dynamic dwell time and varying signal/noise levels, where a fixed-size smoothing window would either leave noise in low-signal sections or clip true peaks in high-signal sections.

## When NOT to use

- Input is not raw MRM ion chromatogram signal but already a peak table or quantitation result — smoothing is not meaningful there.
- The chromatogram has uniform dwell time and homogeneous noise so a fixed-window smoother suffices and the section-adaptive background rule adds no value.
- You are quantitating coeluted or saturated peaks — MRMQuant's separate deconvolution and regression steps handle those cases, not the smoothing step.

## Inputs

- Raw MRM ion chromatogram as retention-time/intensity pairs, extracted from vendor raw data (converted via ProteoWizard msconvert to mzML or MRMQuant's binary MATLAB MAT-file)

## Outputs

- Smoothed chromatogram as retention-time/intensity pairs (e.g. smoothed_chromatogram.csv), one row per point, loadable as a two-column table

## How to apply

Load the raw ion chromatogram (retention time, intensity pairs) from the input file (e.g. mzML-derived data extracted from vendor raw data via ProteoWizard msconvert). Partition the chromatogram into sections and compute each section's background level as its 30th-percentile intensity. Compare each section's signal extent against 1.2 times its background: expand the section if the signal is below 1.2× background or reduce it if above, so each section starts and ends at a signal not higher than 1.2× its estimated background. Apply the section-adaptive smoothing across the chromatogram with sizes of chromatogram sections adjusted for dynamic dwell time, then write retention-time/intensity pairs to a smoothed chromatogram file. Validate that the smoothed profile is loadable as a two-column table and follows the section-adaptive background rule where no true peak is present.

## Related tools

- **MRMQuant** (Implements dynamic signal smoothing over MRM ion chromatograms with dynamic dwell time, using the section-adaptive background rule before peak detection) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime environment in which MRMQuant's smoothing is executed; chromatogram data are held in a binary MATLAB MAT-file)
- **ProteoWizard (msconvert)** (Converts vendor MRM raw data into mzML or MAT-file-compatible formats so the ion chromatograms can be fed into MRMQuant's smoothing)

## Examples

```
python -c "import pandas as pd; x=pd.read_csv('smoothed_chromatogram.csv'); assert x.shape[1]==2; print(x.head())"
```

## Evaluation signals

- Output file exists and loads as a two-column retention-time/intensity table without parse errors.
- In sections with no true peak, the smoothed signal sits at ~1.2× the section's 30th-percentile background, consistent with MRMQuant's dynamic signal smoothing rule.
- Sections start and end at a signal not higher than 1.2 times their estimated background after adjustment.
- No true peak tips are clipped: the smoothing window never shrinks a section so far that a genuine peak's maximum intensity is reduced relative to the raw signal.
- Monotonic retention-time ordering is preserved in the smoothed chromatogram (one point per raw point).

## Limitations

- Requires the section background to be well-estimated by the 30th percentile; if a section is dominated by a real peak, the background estimate may be biased and the expansion/reduction rule could over- or under-smooth.
- The 1.2× threshold and 30th-percentile background are MRMQuant defaults; tuning them for other instruments or dwell-time profiles is outside what the article documents.
- Cannot recover signal lost to vendor-specific acquisition artifacts; it operates on whatever retention-time/intensity pairs msconvert or MAT-file loading yields.

## Evidence

- [other] In MRMQuant's dynamic signal smoothing, each chromatogram section is expanded or reduced so it starts and ends at a signal not higher than 1.2 times the background intensity: [evidence span withheld — anchor status: not_found]
- [other] adjusts the sizes of chromatogram sections used for smoothing windows in MRM data with dynamic dwell time: [evidence span withheld — anchor status: not_found]
