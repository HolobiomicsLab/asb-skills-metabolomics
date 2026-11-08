---
name: multidimensional-feature-matching
description: Use when aligning detected features across LC-IMS-MS/MS samples where
  m/z, drift time, and retention time must all be matched to achieve high-confidence
  feature correspondence between samples.
when_to_use_negative:
- Input data are 1D or 2D only (e.g., simple LC-MS without ion mobility), where DEIMoS's
  multidimensional matching offers no advantage over standard m/z-versus-RT alignment
- Features have already been aligned and only coordinate correction is needed; in
  that case the SVR-fitting sub-step applies but the matching step is redundant
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Provides the alignment module that performs multidimensional feature matching
    across samples using m/z, drift time, and retention time simultaneously
  repo: https://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:6b43cc01489d7433d533ead397819d8b4041acaa8dc9915807b69a95a935a51b
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multidimensional-feature-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches features across study samples by simultaneously considering all acquired dimensions (m/z, drift time, retention time) rather than pairwise 2D comparisons, increasing cross-sample alignment/feature matching confidence in DEIMoS workflows.

## When to use

Use when aligning detected features across LC-IMS-MS/MS samples where m/z, drift time, and retention time must all be matched to achieve high-confidence feature correspondence between samples.

## When NOT to use

- Input data are 1D or 2D only (e.g., simple LC-MS without ion mobility), where DEIMoS's multidimensional matching offers no advantage over standard m/z-versus-RT alignment
- Features have already been aligned and only coordinate correction is needed; in that case the SVR-fitting sub-step applies but the matching step is redundant

## Inputs

- per-sample detected feature tables stored in HDF5 (e.g., example_alignment.h5, key='A') loaded via deimos.load
- per-dimension tolerances (mz_tol=200e-6, dt_tol=0.04, rt_tol)

## Outputs

- cross-sample feature correspondence (matched features across samples)
- fitted SVR coefficients per dimension for coordinate correction
- aligned feature coordinates corrected to the reference frame

## How to apply

Load each sample's detected feature table (e.g., from an Agilent 6560 Ion Mobility LC/Q-TOF acquisition) via deimos.load. Designate an in-study sample as the alignment reference, then match each remaining sample's features to it within per-dimension tolerances: m/z tolerance 200e-6 (ppm), drift time tolerance 0.04, and retention time tolerance derived from observed drift. DEIMoS uses all dimensions simultaneously so that separation between features is greater than in a sequential m/z-versus-RT followed by DT approach, yielding roughly 90% intersection when tolerances comparable to cross-sample alignment are imposed, versus ~13% for pairwise 2D matching. After matching, fit a support vector regression per dimension (RBF kernel) to model nonlinear sample drift relative to the reference, then apply the fitted transforms to correct each sample's feature coordinates.

## Related tools

- **DEIMoS** (Provides the alignment module that performs multidimensional feature matching across samples using m/z, drift time, and retention time simultaneously) — https://github.com/pnnl/deimos

## Examples

```
mz_tol=200e-6, dt_tol=0.04
```

## Evaluation signals

- Intersection of matched features across samples should reach ~90% when imposing the same tolerances used in cross-sample alignment, rather than ~13% from 2D-only matching
- Feature coordinates deviated by an average of 5.8 ppm (m/z), 0.2% (drift time), and 0.05 min (retention time) when comparing 3D processing to m/z versus RT followed by DT; corrected coordinates should fall within these magnitudes of the reference
- Verify that each sample's matched features overlap the reference within the specified mz_tol=200e-6 and dt_tol=0.04 tolerances
- Check that the fitted SVR transform, when applied, reduces residual drift between matched features and the reference

## Limitations

- No quantitative acceptance threshold for alignment quality is reported; validation is qualitative (improved cross-sample feature matching)
- Nonlinear drift (e.g., from a degrading LC column) requires the RBF kernel; a linear model would fail on such samples
- Matching quality depends on tolerances being tight enough to avoid false matches but loose enough to absorb instrument drift; the article does not prescribe a universal tuning recipe

## Evidence

- [methods] Once features are matched between samples, DEIMoS models the relationship between them using support vector regression as implemented in scikit-learn, then applies the fit transform; the RBF kernel can account for nonlinear relationships such as those from a degrading LC column.: [evidence span withheld — anchor status: not_found]
