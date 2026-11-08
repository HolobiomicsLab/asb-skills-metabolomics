---
name: isotopic-signature-detection
description: Use when the input is a DEIMoS-aligned multi-sample feature table (e.g.
  example_alignment.h5) in an N-dimensional space (m/z, drift time, retention time)
  and the goal is to glean ion charge state or identification evidence from isotopic
  signatures.
when_to_use_negative:
- Input is raw instrument data rather than a processed, aligned feature table — isotope
  detection in DEIMoS operates on features, not raw spectra, and the article's demonstration
  loads from example_alignment.h5, not raw files
- The analysis requires peak detection, alignment, or CCS calibration first — those
  are separate DEIMoS steps that must precede isotope detection
- The data are MS2 spectra needing deconvolution — MS/MS spectral deconvolution is
  a distinct DEIMoS functionality, not isotope detection
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: provides the deimos.isotopes N-dimensional isotope detection routine, deimos.load
    for reading the aligned feature table, and deimos.threshold for intensity pre-filtering
  repo: github.com/pnnl/deimos
- name: Python
  role: the API language in which the isotope-detection workflow is written and executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:3fa878b020f77bcd0e28c3a0e8691dd5f6bb72f5b1e9393fb8d7cc620f045249
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# isotopic-signature-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect and annotate isotopologue groups in aligned multi-sample feature tables using DEIMoS's N-dimensional isotope detection, inferring ion charge state from isotopologue spacing and downselecting signatures by m/z error and member count.

## When to use

Use when the input is a DEIMoS-aligned multi-sample feature table (e.g. example_alignment.h5) in an N-dimensional space (m/z, drift time, retention time) and the goal is to glean ion charge state or identification evidence from isotopic signatures.

## When NOT to use

- Input is raw instrument data rather than a processed, aligned feature table — isotope detection in DEIMoS operates on features, not raw spectra, and the article's demonstration loads from example_alignment.h5, not raw files
- The analysis requires peak detection, alignment, or CCS calibration first — those are separate DEIMoS steps that must precede isotope detection
- The data are MS2 spectra needing deconvolution — MS/MS spectral deconvolution is a distinct DEIMoS functionality, not isotope detection

## Inputs

- aligned multi-sample feature table in HDF5 format (e.g. example_alignment.h5) with per-sample keys, containing N-dimensional feature coordinates (m/z, drift time, retention time) and intensities

## Outputs

- detected isotopologue groups annotated with charge state and isotopic signature assignments, saved to e.g. isotopic_signatures.h5

## How to apply

Load the aligned feature table with deimos.load (per-sample keys, e.g. key="A" from example_alignment.h5), then run deimos.isotopes over the N-dimensional feature coordinates, constraining the search with a rough m/z tolerance from the expected isotopic m/z delta (1.003355 Da for C13) plus drift time and retention time tolerances tol=[0.1, 0.7, 0.15]. Downselect candidate isotopologues by requiring a maximum m/z error of 50 ppm relative to the expected isotopic signature, then screen to keep only signatures with at least 3 members as a first quality filter. Infer each monoisotopic feature's ion charge state from the observed isotopologue spacing (max 5 substitutions, max charge 1 in the demonstrated workflow) and annotate each signature accordingly. The N-dimensional search is justified because DEIMoS simultaneously uses all dimensions, which increases feature-matching confidence and mitigates convolution artifacts.

## Related tools

- **DEIMoS** (provides the deimos.isotopes N-dimensional isotope detection routine, deimos.load for reading the aligned feature table, and deimos.threshold for intensity pre-filtering) — github.com/pnnl/deimos
- **Python** (the API language in which the isotope-detection workflow is written and executed)

## Examples

```
sig = deimos.isotopes(deimos.load("example_alignment.h5", key="A"), delta=1.003355, tol=[0.1, 0.7, 0.15], max_substitutions=5, max_charge=1)
```

## Evaluation signals

- Retained signatures satisfy the 50 ppm m/z-error downselection relative to the expected isotopic signature
- Each retained isotopic signature contains at least 3 members (the stated first screening criterion)
- Charge-state assignments are consistent with the observed isotopologue spacing (≤ 5 substitutions, ≤ charge 1 in the demonstrated case)
- Observed m/z spacing between monoisotopic feature and isotopologues matches the expected delta (1.003355 Da for C13) within the applied tolerance

## Limitations

- The demonstrated workflow constrained the maximum number of isotopic substitutions to 5 and maximum charge to 1; heavier charge states or larger isotope clusters require re-tuning these parameters
- Isotope detection is applied after alignment; the method presumes a previously produced aligned multi-sample feature table rather than raw files
- Only isotopic signatures with at least 3 members survive the first screening, so signatures with fewer members are discarded even if chemically real

## Evidence
