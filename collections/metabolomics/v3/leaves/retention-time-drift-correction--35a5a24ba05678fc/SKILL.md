---
name: retention-time-drift-correction
description: Use when per-sample detected feature tables must be aligned to a reference
  sample and coordinates drift between acquisitions (e.g. LC-IMS-MS/MS runs from Bruker
  or Waters instruments).
when_to_use_negative:
- Input is already a single aligned feature table — no cross-sample drift correction
  is needed
- Data are 1D or 2D spectra without an in-study reference sample available for alignment
- The alignment step has not yet matched features between samples; SVR requires matched
  features first
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Provides the deimos.alignment module for reference-based feature matching
    and SVR-based coordinate correction
  repo: github.com/pnnl/deimos
- name: Python
  role: Runtime for the DEIMoS API used to load .h5 feature tables and invoke alignment
  repo: github.com/pnnl/deimos
- name: SVR
  role: scikit-learn support vector regression with RBF kernel used to model nonlinear
    sample drift per dimension
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:3c1cf791a70b82a3535ba38a8c7d691d416e4a1a20bc62ee559ed3066688d56b
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
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# retention-time-drift-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reference-based alignment of multi-dimensional MS feature tables using support vector regression (SVR) to model and correct coordinate drift (m/z, drift time, retention time) between samples, as implemented in DEIMoS.

## When to use

Use when per-sample detected feature tables must be aligned to a reference sample and coordinates drift between acquisitions (e.g. LC-IMS-MS/MS runs from Bruker or Waters instruments).

## When NOT to use

- Input is already a single aligned feature table — no cross-sample drift correction is needed
- Data are 1D or 2D spectra without an in-study reference sample available for alignment
- The alignment step has not yet matched features between samples; SVR requires matched features first

## Inputs

- Per-sample detected feature tables (HDF5, e.g. key='A' from example_alignment.h5) with m/z, drift time, and retention time columns
- An in-study sample designated as the alignment reference
- Alignment tolerances: mz_tol (e.g. 200e-6) and dt_tol (e.g. 0.04)

## Outputs

- Corrected feature coordinates per sample aligned to the reference in m/z, drift time, and retention time
- Fitted SVR coefficients/transforms stored per sample and per dimension

## How to apply

Load per-sample detected feature tables (e.g. the 'A' key from example_alignment.h5) with deimos.load, then designate an in-study sample as the alignment reference. Match each sample's features to that reference within per-dimension tolerances (mz_tol=200e-6, dt_tol=0.04) using the deimos.alignment module. Once features are matched between samples, fit a support vector regression per dimension — m/z, drift time, and retention time — using an RBF kernel, because the RBF kernel accounts for nonlinear relationships such as those arising from a degrading LC column. Apply the fitted SVR transforms to correct each sample's feature coordinates and store the fitted coefficients. Validation relies on confirming that the corrected feature coordinates overlap the reference within the alignment tolerances; no quantitative acceptance threshold is reported in the paper, so accept based on qualitatively improved cross-sample feature matching.

## Related tools

- **DEIMoS** (Provides the deimos.alignment module for reference-based feature matching and SVR-based coordinate correction) — github.com/pnnl/deimos
- **Python** (Runtime for the DEIMoS API used to load .h5 feature tables and invoke alignment) — github.com/pnnl/deimos
- **SVR** (scikit-learn support vector regression with RBF kernel used to model nonlinear sample drift per dimension)

## Examples

```
import deimos

# Load the per-sample detected feature table
data = {}
data["A"] = deimos.load("example_alignment.h5", key="A")

# Fit a support vector regression for each dimension (m/z, drift time,
# retention time) using an RBF kernel, then apply the fitted transform
corrected, coeffs = deimos.alignment.fit_A(data, mz_tol=200e-6, dt_tol=0.04)
```

## Evaluation signals

- Corrected feature coordinates should overlap the reference within the applied tolerances (mz_tol=200e-6, dt_tol=0.04)
- Cross-sample feature matching confidence improves after applying the SVR transform compared with uncorrected coordinates
- Fitted SVR coefficients exist for each dimension (m/z, drift time, retention time) and are persisted for reuse
- Reference sample itself is unchanged — its coordinates serve as the target of the fit
- The RBF kernel is used, consistent with handling nonlinear relationships such as LC column degradation

## Limitations

- No quantitative acceptance threshold is reported in the paper; validation is qualitative (improved cross-sample feature matching)
- Matching must occur before fitting; the RBF kernel is chosen to capture nonlinear relationships (e.g. degrading LC column), so a linear model may undercorrect
- Per-dimension tolerances (mz_tol, dt_tol) must be set manually and appropriate for the instrument and acquisition

## Evidence

- [methods] Once features are matched between samples, DEIMoS models the relationship between them using support vector regression (SVR) as implemented in scikit-learn: [evidence span withheld — anchor status: not_found]
