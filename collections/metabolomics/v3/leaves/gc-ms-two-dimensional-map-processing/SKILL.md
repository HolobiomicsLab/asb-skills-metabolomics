---
name: gc-ms-two-dimensional-map-processing
description: Use when the input is GC–MS data (e.g. aroma odor or human breath) represented
  as a two-dimensional MS map of m/z vs retention time, and the goal is to identify
  multivariate chemo-/biomarker features at very low concentration levels (down to
  parts per billion) without peak picking.
when_to_use_negative:
- When the data are not chromatography–MS (e.g. direct-infusion spectra with no m/z
  vs retention-time map).
- When a validated conventional peak picking workflow is already in place and the
  user cannot accept NPFimg's image-based marker features instead of picked peaks.
- When data dimensionality differs from the two-dimensional m/z vs retention-time
  map assumed by NPFimg.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NPFimg
  role: Core tool that processes the two-dimensional MS map (m/z vs retention time)
    via image processing and machine learning to identify and visualize marker features
    without peak picking.
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: Widely used baseline peak-based workflow against which NPFimg's signal-acquisition
    and marker-identification error rates are compared.
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_content_hash: sha256:8eb567dbece2834db881f23bd5eaaebeb1dce689707080107e5fa9a1fa56d16c
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gc-ms-two-dimensional-map-processing

## Summary

NPFimg-style processing of a two-dimensional GC–MS map (m/z vs retention time) that combines image processing and machine learning to identify multivariate chemo-/biomarker features without the conventional peak picking process, which suffers from false peak detections.

## When to use

Use when the input is GC–MS data (e.g. aroma odor or human breath) represented as a two-dimensional MS map of m/z vs retention time, and the goal is to identify multivariate chemo-/biomarker features at very low concentration levels (down to parts per billion) without peak picking.

## When NOT to use

- When the data are not chromatography–MS (e.g. direct-infusion spectra with no m/z vs retention-time map).
- When a validated conventional peak picking workflow is already in place and the user cannot accept NPFimg's image-based marker features instead of picked peaks.
- When data dimensionality differs from the two-dimensional m/z vs retention-time map assumed by NPFimg.

## Inputs

- Gas chromatography–mass spectrometry (GC–MS) data, e.g. aroma odor or human breath measurements
- Two-dimensional MS map (m/z vs retention time) built from the raw GC–MS data
- Liquid chromatography–MS data (stated as potentially applicable)

## Outputs

- Multivariate chemo-/biomarker (marker) features identified in the 2D MS map
- Visualizations of the identified marker features on the two-dimensional MS map
- Signal-acquisition and marker-identification error-rate metrics for comparison with XCMS

## How to apply

Convert the chromatography–MS data into a two-dimensional MS map with m/z on one axis and retention time on the other; NPFimg then processes this image to discriminate analytes. Instead of running a conventional peak picking step (e.g. as in XCMS), apply NPFimg's combined image-processing and machine-learning pipeline to the map to locate multivariate marker features, and visualize them directly on the 2D map. NPFimg was benchmarked against the widely used XCMS and reported lower error rates of both signal acquisition and marker identification, which justifies skipping peak picking. The demonstration datasets are aroma odor and human breath GC–MS at parts-per-billion concentrations; the same approach is stated to be potentially applicable to liquid chromatography–MS and diverse metabolomics/chemometrics data.

## Related tools

- **NPFimg** (Core tool that processes the two-dimensional MS map (m/z vs retention time) via image processing and machine learning to identify and visualize marker features without peak picking.) — http://github.com/poomcj/NPFimg
- **XCMS** (Widely used baseline peak-based workflow against which NPFimg's signal-acquisition and marker-identification error rates are compared.)

## Examples

```
git clone http://github.com/poomcj/NPFimg && python npfimg.py --ms-map aroma_odor_2d_map — see https://pubs.acs.org/doi/full/10.1021/acs.analchem.1c03163 for the full manual script; no concrete CLI is given in the README beyond the repository URL.
```

## Evaluation signals

- Marker identification is demonstrated on the aroma odor and human breath GC–MS case studies even at the parts per billion level — check that marker features are recovered at these concentrations.
- Compare error rates of signal acquisition and marker identification against XCMS; NPFimg should show lower error rates.
- Verify that no conventional peak picking step was required — marker features are found directly from the 2D map by image processing.
- Check that identified marker features are multivariate (not single ions) and can be visualized on the m/z vs retention-time map.

## Limitations

- The published study shows only limited applications (aroma odor and human breath GC–MS); broader applicability to diverse metabolomics/chemometrics data remains potential rather than demonstrated.
- The README notes that further details were to be updated later, so the manual/detailed documentation may be incomplete (the full manual is referenced via the article DOI).
- Conventional peak picking suffers from false peak detections; NPFimg's alternative has been validated mainly against XCMS, so error rates should be checked for the specific instrument and dataset.

## Evidence
