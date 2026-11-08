---
name: chromatographic-peak-baseline-reconstruction
description: Use when MRM ion chromatograms contain noisy peaks, unevenly sampled
  signals, or coeluted isomers where connecting nearby valleys (e.g., TargetLynx)
  misestimates the baseline; adaptive section-wise background reconstruction is needed
  before peak integration.
when_to_use_negative:
- The input is already a de-noised, baseline-corrected feature table rather than raw
  ion chromatograms — smoothing would distort valid peak shapes.
- The goal is a batch-level normalization across samples rather than per-chromatogram
  baseline reconstruction; use the separate batch-effect correction step in MRMQuant
  instead.
- Coeluted isomer peaks need to be resolved into separate transitions rather than
  baseline-adjusted — use MRMQuant's automatic deconvolution of coeluted peaks feature
  instead.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Implements the dynamic signal smoothing / adaptive baseline reconstruction
    applied to MRM ion chromatograms before peak detection and quantitation.
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime environment in which MRMQuant runs; the built-in function findpeaks
    is used for peak detection on the smoothed chromatograms.
- name: ProteoWizard
  role: msconvert is used to convert vendor raw MRM data to mzML or mzXML before loading
    into MRMQuant for baseline reconstruction.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:0d28a3cf8f65600d416e31d9eef4c8e84d6860ae9de4d1d48eb43603b54fc78a
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatographic-peak-baseline-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill describing MRMQuant's dynamic signal smoothing, in which each ion chromatogram section is expanded or reduced until its signal extent equals 1.2× its background intensity, with background determined as the 30th percentile of the section's signal — this adaptive baseline reconstruction underlies accurate peak integration in MRM targeted metabolomics quantitation.

## When to use

Use when MRM ion chromatograms contain noisy peaks, unevenly sampled signals, or coeluted isomers where connecting nearby valleys (e.g., TargetLynx) misestimates the baseline; adaptive section-wise background reconstruction is needed before peak integration.

## When NOT to use

- The input is already a de-noised, baseline-corrected feature table rather than raw ion chromatograms — smoothing would distort valid peak shapes.
- The goal is a batch-level normalization across samples rather than per-chromatogram baseline reconstruction; use the separate batch-effect correction step in MRMQuant instead.
- Coeluted isomer peaks need to be resolved into separate transitions rather than baseline-adjusted — use MRMQuant's automatic deconvolution of coeluted peaks feature instead.

## Inputs

- Raw MRM ion chromatogram (retention time, intensity pairs) per targeted transition
- MRM sample file in mzML or mzXML format (converted from vendor raw data using msconvert in ProteoWizard)
- Method file with information about the targeted compounds (e.g., specified RTs and transitions)

## Outputs

- Smoothed chromatogram with adaptive section-wise baseline (retention-time/intensity pairs)
- Baseline estimates per section (1.2× the 30th-percentile background where no true peak is present)
- Peak abundance values derived from the smoothed profile for downstream quantitation

## How to apply

Load the raw ion chromatogram (retention time, intensity pairs) from the MRM sample file (mzML or mzXML converted via ProteoWizard msconvert) and partition it into sections. For each section, compute the background intensity as its 30th percentile signal. Compare each section's signal extent against its estimated background: expand the section if the signal is below 1.2× background or reduce it if above, until each section spans 1.2× its background where no true peak is present. Apply the adaptive background-adjusted smoothing across the chromatogram using the section-adaptive rule before peak detection (MATLAB findpeaks) and quantitation. The rationale is that a fixed valley-connecting baseline overestimates the background at coeluted peaks and underestimates abundance, whereas per-section 30th-percentile backgrounds adapt to local noise and coelution levels in the MRM data with dynamic dwell time.

## Related tools

- **MRMQuant** (Implements the dynamic signal smoothing / adaptive baseline reconstruction applied to MRM ion chromatograms before peak detection and quantitation.) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime environment in which MRMQuant runs; the built-in function findpeaks is used for peak detection on the smoothed chromatograms.)
- **ProteoWizard** (msconvert is used to convert vendor raw MRM data to mzML or mzXML before loading into MRMQuant for baseline reconstruction.)

## Examples

```
MRMQuant (MATLAB); e.g., load MRM sample mzML files and method file, then run MRMQuant's dynamic signal smoothing to produce smoothed chromatograms before findpeaks-based peak detection.
```

## Evaluation signals

- The smoothed intensity profile follows the section-adaptive background rule: each section's smoothed signal equals 1.2× its 30th-percentile background where no true peak is present.
- Peak abundances computed on the smoothed chromatogram show low standard deviations (<0.1) and RT differences across replicate QC samples (e.g., the human aging phospholipid QC set with 184 phospholipids).
- At coeluted isomers (e.g., three PC 34:3 isomers at RTs 4.72, 4.84, and 4.92 min), the reconstructed baseline does not attribute all coeluted peaks to one compound and does not overestimate the background at the targeted compound's valley.
- The output is a loadable two-column retention-time/intensity table, consistent with downstream quantitation in MRMQuant.
- Compared with a valley-connecting baseline (TargetLynx style), the reconstructed baseline yields abundances that are not underestimated due to high coelution in the right valley.

## Limitations

- MRMkit and Skyline exhibited problematic quantitation of coeluted peaks, noisy peaks, and peaks with unevenly sampled signals despite low average STD and RT differences — baseline reconstruction alone does not resolve these failure modes without deconvolution.
- OpenMS was unable to consistently quantify the same peak when multiple peaks existed in the neighborhood of the specified RT, leading to large variations in computed peak abundances — a risk when baseline reconstruction is not coupled with consistent peak-picking.
- TargetLynx achieved the smallest average STD but had the highest number of unquantifiable compounds among the tools, showing that low per-peak variance alone does not guarantee correct baseline reconstruction across all compounds.

## Evidence

- [abstract] MRMQuant possesses the following features to ensure accurate quantitation: (1) dynamic signal smoothing, (2) automatic deconvolution of coeluted peaks, (3) absolute quantitation via standard curves: "MRMQuant possesses the following features to ensure accurate quantitation: (1) dynamic signal smoothing, (2) automatic deconvolution of coeluted peaks"
