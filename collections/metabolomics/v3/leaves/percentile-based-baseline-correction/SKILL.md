---
name: percentile-based-baseline-correction
description: Use when smoothing MRM ion chromatograms (especially with dynamic dwell
  time) where baseline/background intensity must be estimated per-section from raw
  RT-intensity pairs before peak tip detection or peak quantitation.
when_to_use_negative:
- When a single global baseline is sufficient and coeluted peaks, noisy peaks, or
  unevenly sampled signals are not present — MRMQuant's dynamic smoothing is designed
  specifically for those cases, whereas simpler tools (e.g. TargetLynx connecting
  nearby valleys) mishandle them.
- When the input is already a peak-picked abundance table rather than raw RT/intensity
  chromatogram pairs — baseline correction operates on the raw signal, not on downstream
  quantitation results.
- When chromatograms come from untargeted/full-scan acquisitions — the method is described
  for high-throughput LC-tandem MS MRM targeted metabolomics data.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MRMQuant
  role: Implements dynamic signal smoothing with the 30th-percentile background and
    1.2× section-adjustment rule on MRM ion chromatograms; runs as a MATLAB-based
    pipeline before findpeaks-based peak detection
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: 'Implementation environment: MRMQuant generates a binary MATLAB MAT-file that
    is loaded in, and the ''findpeaks'' built-in function is used downstream of smoothing'
- name: msconvert (ProteoWizard)
  role: Converts vendor raw MRM data into mzML or other formats that MRMQuant can
    load prior to smoothing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:8e6678105219a31365282a6c076bbdea33452ed59630c2f2d0e4704531348758
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
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# percentile-based-baseline-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates the background intensity of ion chromatogram sections in targeted MRM metabolomics data using a percentile-of-signal rule (30th percentile), then adaptively expands or trims each section so it starts and ends at a signal not higher than 1.2× background — the basis of MRMQuant's dynamic signal smoothing.

## When to use

Use when smoothing MRM ion chromatograms (especially with dynamic dwell time) where baseline/background intensity must be estimated per-section from raw RT-intensity pairs before peak tip detection or peak quantitation.

## When NOT to use

- When a single global baseline is sufficient and coeluted peaks, noisy peaks, or unevenly sampled signals are not present — MRMQuant's dynamic smoothing is designed specifically for those cases, whereas simpler tools (e.g. TargetLynx connecting nearby valleys) mishandle them.
- When the input is already a peak-picked abundance table rather than raw RT/intensity chromatogram pairs — baseline correction operates on the raw signal, not on downstream quantitation results.
- When chromatograms come from untargeted/full-scan acquisitions — the method is described for high-throughput LC-tandem MS MRM targeted metabolomics data.

## Inputs

- Raw MRM ion chromatogram as retention-time/intensity pairs (from mzML or MRM sample files converted with msconvert/ProteoWizard)
- MRM sample file plus method file describing targeted compounds (precursor/product m/z, expected RT)

## Outputs

- Background-corrected (smoothed) ion chromatogram: RT/intensity pairs with each section's signal bounded by 1.2× its 30th-percentile background
- Section boundaries adjusted so each section starts and ends at a signal not higher than 1.2× background intensity

## How to apply

Load the raw ion chromatogram as retention-time/intensity pairs (e.g. from an mzML-derived data table after msconvert conversion in ProteoWizard). Partition the chromatogram into sections and compute each section's background level as the 30th percentile of its signal intensities. Compare each section's signal extent against its estimated background: expand the section if the signal is below 1.2× background, or reduce it if above, until each section spans 1.2× its 30th-percentile background where no true peak is present. This section-adaptive background rule then feeds MRMQuant's dynamic signal smoothing, which precedes the MATLAB 'findpeaks' function used to locate possible peak tip locations for targeted compounds. Implement in MATLAB (MRMQuant's implementation language) so the smoothed chromatograms can flow directly into the quantitation pipeline.

## Related tools

- **MRMQuant** (Implements dynamic signal smoothing with the 30th-percentile background and 1.2× section-adjustment rule on MRM ion chromatograms; runs as a MATLAB-based pipeline before findpeaks-based peak detection) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Implementation environment: MRMQuant generates a binary MATLAB MAT-file that is loaded in, and the 'findpeaks' built-in function is used downstream of smoothing)
- **msconvert (ProteoWizard)** (Converts vendor raw MRM data into mzML or other formats that MRMQuant can load prior to smoothing)

## Examples

```
matlab -batch "load('sample.mat'); smoothed = MRMQuant_dynamicSmooth(chrom, 'bgPercentile', 30, 'bgFactor', 1.2); writetable(array2table(smoothed), 'smoothed_chromatogram.csv')"
```

## Evaluation signals

- The smoothed intensity profile follows the section-adaptive background rule: each section's smoothed signal equals 1.2× its 30th-percentile background where no true peak is present.
- Each chromatogram section starts and ends at a signal not higher than 1.2 times the background intensity after adjustment.
- Output remains a valid two-column RT/intensity table that downstream peak detection (findpeaks) can consume.
- Isomers of the same transition (e.g. PC 34:3 isomers at RT 4.72, 4.84, 4.92 min in the tested chromatogram) remain resolvable rather than being merged into one baseline region.
- Peak abundances after smoothing show low STD and RT differences across replicates (MRMQuant-style comparison against tools like MRMkit/Skyline reported <0.1 average STD).

## Limitations

- Designed for MRM data with dynamic dwell time; the section sizes and 1.2× thresholds are calibrated for that regime and may not transfer directly to other acquisition types.
- MRMkit and Skyline had problematic quantitation of coeluted peaks, noisy peaks, and peaks with unevenly sampled signals — tools without this adaptive background rule risk the same failure modes; conversely, using this skill alone does not fix batch effects or absolute quantitation, which MRMQuant handles in separate steps.
- The 30th-percentile background is an approximation of the baseline in each section; a section dominated by a broad true peak may have its background overestimated, requiring the section to be re-partitioned or reduced.

## Evidence

- [results] each chromatogram section is expanded or reduced so it starts and ends at a signal not higher than 1.2 times the background intensity: [evidence span withheld — anchor status: not_found]
- [abstract] findpeaks used after smoothing: [evidence span withheld — anchor status: not_found]
- [results] dynamic dwell time context: [evidence span withheld — anchor status: not_found]
