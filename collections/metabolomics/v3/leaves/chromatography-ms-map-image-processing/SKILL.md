---
name: chromatography-ms-map-image-processing
description: Use when raw 2D chromatography–MS maps (m/z vs retention time, GC–MS
  or LC–MS) need analyte discrimination and marker-feature identification/visualization
  and conventional peak picking is unreliable or causes false peak detections.
when_to_use_negative:
- Input is already a peak-picked feature table or a processed feature matrix rather
  than a raw 2D MS map image.
- The task requires the conventional peak picking workflow itself (e.g. when XCMS-style
  peak detection outputs are explicitly needed) instead of the peak-picking-free image
  approach.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NPFimg
  role: 'Primary tool: processes the 2D MS map (m/z vs retention time) via image processing
    + machine learning to discriminate analytes and identify/visualize marker features'
  repo: github.com/poomcj/NPFimg
- name: XCMS
  role: Comparison benchmark used to show NPFimg has lower error rates of signal acquisition
    and marker identification
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:0e0ada1b14063cf0b58d97dc7aacef48cd77dd2c97681ac6fd2f319453ba81cb
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chromatography-ms-map-image-processing

## Summary

Analyzes two-dimensional chromatography–MS maps (m/z vs retention time) with NPFimg, which combines image processing and machine learning to discriminate analytes and identify/visualize chemo-/biomarker features without conventional peak picking.

## When to use

Use when raw 2D chromatography–MS maps (m/z vs retention time, GC–MS or LC–MS) need analyte discrimination and marker-feature identification/visualization and conventional peak picking is unreliable or causes false peak detections.

## When NOT to use

- Input is already a peak-picked feature table or a processed feature matrix rather than a raw 2D MS map image.
- The task requires the conventional peak picking workflow itself (e.g. when XCMS-style peak detection outputs are explicitly needed) instead of the peak-picking-free image approach.

## Inputs

- Two-dimensional chromatography–MS map (m/z vs retention time intensity image), e.g. from GC–MS runs
- Case-study datasets such as aroma odor and human breath GC–MS data (example data available in the NPFimg repository)

## Outputs

- Discriminated analyte assignments from the machine-learning stage
- Identified multivariate chemo-/biomarker features
- Marker-feature visualization image showing the identified features on the two-dimensional MS map

## How to apply

Load the chromatography–MS run as a two-dimensional MS map indexed by m/z and retention time and treat it as an intensity image for NPFimg. Apply the image-processing stage to comprehensively characterize the signals in the MS map, deliberately skipping the conventional peak picking process, which suffers from false peak detections. Then apply the machine-learning stage to the processed map image to discriminate analytes and automatically identify multivariate chemo-/biomarker features. Generate NPFimg's marker-feature visualization showing the identified features on the 2D MS map and save it as the output. NPFimg was demonstrated on GC–MS data for aroma odor and human breath case studies even at the parts per billion level, and is reported as potentially applicable to diverse metabolomics/chemometrics using GC–MS and liquid chromatography–MS.

## Related tools

- **NPFimg** (Primary tool: processes the 2D MS map (m/z vs retention time) via image processing + machine learning to discriminate analytes and identify/visualize marker features) — github.com/poomcj/NPFimg
- **XCMS** (Comparison benchmark used to show NPFimg has lower error rates of signal acquisition and marker identification)

## Evaluation signals

- Confirm the marker-feature visualization image is produced and the identified features overlay sensibly on the 2D MS map (m/z vs retention time).
- Check that analyte discrimination does not rely on a peak-picked peak list, i.e. the pipeline operates directly on the map image without conventional peak picking.
- Validate that the method captures analyte signals at low concentration (the paper demonstrates parts-per-billion-level marker identification on GC–MS).
- Where a comparison to XCMS is possible, verify that signal-acquisition and marker-identification error rates are lower for NPFimg than for XCMS.

## Limitations

- Conventional peak picking is avoided because it suffers from false peak detections; errors may still arise in signal acquisition and marker identification.
- The published study shows limited applications (aroma odor and human breath GC–MS); broader applicability to other metabolomics/chemometrics data is asserted as potential rather than fully demonstrated.
- The repository README is incomplete — additional usage details were deferred ('I will update soon').

## Evidence
