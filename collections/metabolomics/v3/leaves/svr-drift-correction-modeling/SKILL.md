---
name: svr-drift-correction-modeling
description: Use when detected features from LC-IMS-MS/MS samples must be aligned
  to a reference sample and coordinates for m/z, drift time, and retention time drift
  nonlinearly between runs.
when_to_use_negative:
- Input data are already aligned feature tables with no cross-sample coordinate drift
  to correct
- The feature matching step (before SVR fitting) has not been performed — DEIMoS aligns
  features between samples first, then models the relationship with SVR
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Python API used to load feature tables, match features to the reference within
    tolerances, fit per-dimension SVR with RBF kernel, and apply the correction transform
  repo: github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:10a55aa2f4af7e0229fd8f922a1cd9f77c4bd078b70a49dd19c61528570a2f08
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
  - SVR
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# svr-drift-correction-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit per-dimension support vector regression models to correct feature coordinate drift between LC-IMS-MS/MS samples relative to an in-study reference, using an RBF kernel to capture nonlinear drift such as that from a degrading LC column.

## When to use

Use when detected features from LC-IMS-MS/MS samples must be aligned to a reference sample and coordinates for m/z, drift time, and retention time drift nonlinearly between runs.

## When NOT to use

- Input data are already aligned feature tables with no cross-sample coordinate drift to correct
- The feature matching step (before SVR fitting) has not been performed — DEIMoS aligns features between samples first, then models the relationship with SVR

## Inputs

- Per-sample detected feature tables (e.g. key 'A' from example_alignment.h5) with coordinates for m/z, drift time, and retention time
- Designated in-study reference sample feature table
- Alignment tolerance parameters: mz_tol=200e-6, dt_tol=0.04

## Outputs

- Corrected feature coordinate tables for each sample aligned to the reference
- Stored fitted SVR coefficients per dimension (m/z, drift time, retention time)

## How to apply

Load each sample's detected feature table (e.g. key 'A' from example_alignment.h5) with deimos.load and designate an in-study sample as the alignment reference. Match each sample's features to the reference within per-dimension tolerances (m/z tolerance 200e-6, drift time tolerance 0.04) using the deimos.alignment module. Fit a support vector regression per dimension (m/z, drift time, retention time) using the radial basis function (RBF) kernel, which accounts for nonlinear relationships such as those from a degrading LC column. Apply the fitted SVR transforms to correct each sample's feature coordinates and store the fitted coefficients for downstream cross-sample feature matching.

## Related tools

- **DEIMoS** (Python API used to load feature tables, match features to the reference within tolerances, fit per-dimension SVR with RBF kernel, and apply the correction transform) — github.com/pnnl/deimos

## Examples

```
from deimos.alignment import fit; fit(sample, reference, mz_tol=200e-6, dt_tol=0.04, kernel='RBF')
```

## Evaluation signals

- Corrected feature coordinates for each sample overlap the reference within the alignment tolerances (mz_tol=200e-6, dt_tol=0.04); no quantitative acceptance threshold is reported, so accept based on qualitatively improved cross-sample feature matching
- Fitted SVR coefficients are stored and can be re-applied to new sample tables
- Per-dimension fits cover all three coordinates (m/z, drift time, retention time) rather than only m/z versus retention time
- Cross-sample feature matching intersection increases substantially (to ~90%) after correction, consistent with the reported improvement when imposing alignment tolerances

## Limitations

- No quantitative acceptance threshold is reported in the paper for validating the corrected coordinates; validation is qualitative
- The RBF kernel must be used to account for nonlinear drift; a linear model would not capture degradation of the LC column
- Drift correction depends on the quality of the initial feature matching to the reference within the specified tolerances

## Evidence
