---
name: replicate-feature-consistency-checking
description: Use when features have been detected per-sample (e.g. in positive and
  negative ionization mode LC-IMS-MS/MS data) from study samples acquired in triplicate
  and you need to downselect only those features reproducibly observed in all replicate
  injections.
when_to_use_negative:
- Study samples were acquired as single injections rather than analytical triplicates
  — there is no replicate set to check consistency against.
- Input is already an aligned cross-sample feature table rather than per-sample detected
  feature coordinates.
- The goal is exploratory peak picking on blanks or QC samples rather than confident
  feature calls in study samples.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Used to detect features per sample (with deimos.threshold at 500 pre-detection
    and 1000 post-detection) and to load the resulting per-mode HDF5 feature coordinate
    tables that are then intersected across analytical triplicates.
  repo: github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:16bb88dcebaa38d3a2b9e07d1d8f5dbb8c916646b2abbb46e0831e7b1afda51e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Replicate feature consistency checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation practice from the DEIMoS LC-IMS-MS/MS workflow in which detected features are required to be observed across analytical triplicate injections of study samples, providing additional confidence in detected features beyond local-maximum detection and intensity thresholds.

## When to use

Use when features have been detected per-sample (e.g. in positive and negative ionization mode LC-IMS-MS/MS data) from study samples acquired in triplicate and you need to downselect only those features reproducibly observed in all replicate injections.

## When NOT to use

- Study samples were acquired as single injections rather than analytical triplicates — there is no replicate set to check consistency against.
- Input is already an aligned cross-sample feature table rather than per-sample detected feature coordinates.
- The goal is exploratory peak picking on blanks or QC samples rather than confident feature calls in study samples.

## Inputs

- Per-sample feature coordinate tables (m/z, drift time, retention time, intensity) exported as HDF5 by DEIMoS, one table per ionization mode
- Study sample mzML(.gz) files acquired in analytical triplicate (e.g. Agilent 6560 Ion Mobility LC/Q-TOF data from MassIVE MSV000091746)

## Outputs

- Downselected feature coordinate tables containing only features observed in all analytical triplicate injections, per ionization mode
- A confidence-filtered feature list suitable for downstream DEIMoS steps (alignment, CCS calibration, isotope detection, MS/MS deconvolution)

## How to apply

First detect features independently in each mzML(.gz) file from the MassIVE deposit (MSV000091746, Agilent 6560 example data), applying the pre-detection nominal intensity threshold of 500 and the post-detection threshold of 1000 via deimos.threshold, and saving per-mode feature coordinate tables (m/z, drift time, retention time, intensity) as HDF5. Then, rather than treating every locally maximal ion coordinate as a feature, require that a given feature be observed across all analytical triplicate injections of each study sample (the paper acquired study samples in triplicate, e.g. 9 Bruker study samples and 6 Waters study samples in triplicate). The rationale is that replicate observation distinguishes real analyte signals from noise or sporadic artifacts that survive intensity filtering. Apply this check separately for positive and negative ionization modes and retain only features present in the full triplicate set. This replicate-consistency requirement was the additional confidence criterion imposed by DEIMoS's feature detection workflow in the article.

## Related tools

- **DEIMoS** (Used to detect features per sample (with deimos.threshold at 500 pre-detection and 1000 post-detection) and to load the resulting per-mode HDF5 feature coordinate tables that are then intersected across analytical triplicates.) — github.com/pnnl/deimos

## Examples

```
ms1 = deimos.load("example_data.h5", key="ms1"); ms1_peaks = deimos.threshold(ms1_peaks, threshold=1000)
```

## Evaluation signals

- Retained features must be present in all three replicate injections of each study sample; any feature missing from at least one replicate is dropped.
- The check is performed separately in positive and negative ionization modes, yielding two independent downselected feature tables.
- Feature coordinate tables retain the m/z, drift time, retention time, intensity schema produced by deimos.load of the .h5 files.
- Features that pass should also survive the post-detection intensity threshold of 1000 applied by deimos.threshold.
- Downstream DEIMoS alignment should show improved feature-matching confidence among datasets, consistent with the paper's claim that multidimensional processing increases alignment/feature matching confidence.

## Limitations

- The replicate requirement assumes study samples were actually acquired in analytical triplicate; the paper reports 912 data files for 112 study samples, 40 QC samples, and 10 blanks collected at 3 (replicates), but datasets with fewer replicates cannot support this check.
- Requiring observation in all triplicates is a strict confidence criterion and may discard low-abundance features that are detected in fewer than three replicates.
- This check is downstream of DEIMoS's intensity thresholds (500 pre-detection, 1000 post-detection) and local-maximum peak detection; it does not replace them.

## Evidence

- [methods] To provide additional confidence in detected features, DEIMoS required that a given feature be observed across analytical triplicates.: [evidence span withheld — anchor status: not_found]
