---
name: reference-based-feature-alignment
description: Use when per-sample detected feature tables (e.g., key 'A' from example_alignment.h5)
  show coordinate drift between LC-IMS-MS/MS samples and features must be matched
  across samples before downstream quantification.
when_to_use_negative:
- Input is already an aligned, cross-sample feature table — re-alignment would distort
  quantified intensities.
- The study lacks a stable in-study reference sample (e.g., only a single sample is
  available), since SVR fitting requires matched pairs against a reference.
- The data are not multi-dimensional (e.g., m/z versus RT only with no drift time),
  where the RBF-kernel SVR gains described in the paper's 3D processing would not
  apply.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Provides the deimos.alignment module used to match features to a reference
    within per-dimension tolerances and fit/apply SVR transforms for coordinate correction.
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Runtime environment in which DEIMoS is invoked as an API for loading tables,
    fitting SVR (scikit-learn), and storing corrected outputs.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:1d858896351e6d50881cbf17d8c0156c39407f257339db44cb4f21665060f06d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  repo_url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  - SVR
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-based-feature-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects cross-sample feature coordinate drift in high-dimensional mass spectrometry data by matching each sample's features to an in-study reference and fitting support vector regression (SVR) per dimension (m/z, drift time, retention time), as implemented in DEIMoS.

## When to use

Use when per-sample detected feature tables (e.g., key 'A' from example_alignment.h5) show coordinate drift between LC-IMS-MS/MS samples and features must be matched across samples before downstream quantification.

## When NOT to use

- Input is already an aligned, cross-sample feature table — re-alignment would distort quantified intensities.
- The study lacks a stable in-study reference sample (e.g., only a single sample is available), since SVR fitting requires matched pairs against a reference.
- The data are not multi-dimensional (e.g., m/z versus RT only with no drift time), where the RBF-kernel SVR gains described in the paper's 3D processing would not apply.

## Inputs

- Per-sample detected feature tables with m/z, drift time, retention time and intensity columns (e.g., deimos.load('example_alignment.h5', key='A'))
- A designated in-study reference sample's feature table

## Outputs

- Aligned feature tables with corrected m/z, drift time, and retention time coordinates per sample
- Fitted SVR coefficients/transforms stored for reproducibility
- Cross-sample feature matches with their assigned reference feature IDs

## How to apply

Load per-sample feature tables with deimos.load and designate one in-study sample as the alignment reference. Match each sample's features to the reference within per-dimension tolerances (mz_tol=200e-6, dt_tol=0.04) using the deimos.alignment module. Fit a support vector regression per dimension (m/z, drift time, retention time) using the radial basis function (RBF) kernel, which can model nonlinear relationships such as those from a degrading LC column. Apply the fitted SVR transforms to correct each sample's feature coordinates and store the fitted coefficients. Validate by confirming corrected coordinates overlap the reference within the alignment tolerances; the multidimensional matching increases feature-matching confidence compared with sequential low-dimensional alignment.

## Related tools

- **DEIMoS** (Provides the deimos.alignment module used to match features to a reference within per-dimension tolerances and fit/apply SVR transforms for coordinate correction.) — https://github.com/pnnl/deimos
- **Python** (Runtime environment in which DEIMoS is invoked as an API for loading tables, fitting SVR (scikit-learn), and storing corrected outputs.)

## Examples

```
import deimos; data = deimos.load('example_alignment.h5', key='A'); aligned = deimos.alignment.align(data, ref=data['reference'], mz_tol=200e-6, dt_tol=0.04)
```

## Evaluation signals

- Corrected feature coordinates in each sample overlap the reference features within the alignment tolerances (m/z within ~200 ppm, drift time within 0.04).
- Fitted SVR coefficients are persisted and can be re-applied to reproduce the corrected tables.
- Cross-sample feature-matching confidence improves over unaligned data — e.g., intersection of feature lists between samples increases substantially (~90% when imposing the same tolerances used in cross-sample alignment, per the paper's low-dimensional comparison).
- Residual coordinate deviation after correction is qualitatively reduced relative to the ~5.8 ppm / 0.2% / 0.05 min average deviation reported when comparing 3D processing to m/z versus RT followed by DT.

## Limitations

- No quantitative acceptance threshold for alignment quality is reported in the paper; accept based on qualitatively improved cross-sample feature matching.
- The paper notes few open-source solutions are positioned to fully leverage the latest instrumentation in metabolomics and exposomics data processing, so SVR-based alignment may not generalize to all instrument platforms without retuning tolerances.
- Drift-time tolerance and m/z ppm tolerance (200e-6, 0.04) were demonstrated on Agilent 6560 Ion Mobility LC/Q-TOF data; other instruments may require different values.

## Evidence

- [results] SVR with RBF kernel models nonlinear drift such as from a degrading LC column: [evidence span withheld — anchor status: not_found]
