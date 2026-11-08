---
name: npfimg-marker-feature-identification
description: Use when processing GC–MS or LC–MS chromatography–mass spectrometry data
  as a two-dimensional MS map (m/z vs retention time) to identify and visualize multivariate
  chemo-/biomarker features without conventional peak picking.
when_to_use_negative:
- When the goal is conventional peak detection/alignment via peak-picking tools such
  as XCMS and a peak table is the desired intermediate — NPFimg deliberately avoids
  the conventional peak picking process.
- When the input is not a two-dimensional MS map (m/z vs retention time) — e.g. non-chromatographic
  direct-infusion or NMR spectra — the image-based map approach does not apply.
- When data are single-channel 1D spectra without a retention-time axis, since NPFimg's
  discrimination and marker identification operate on 2D MS maps.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NPFimg
  role: 'Primary tool: combines image processing and machine learning on the 2D MS
    map (m/z vs retention time) to discriminate analytes, identify, and visualize
    multivariate chemo-/biomarker features without conventional peak picking.'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: 'Benchmark comparator: widely used peak-picking tool against which NPFimg''s
    error rates of signal acquisition and marker identification were evaluated; XCMS
    represents the conventional approach NPFimg replaces.'
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  source_content_hash: sha256:01f361b154624291ed522c4c9dc832754c3299120afb2ef107a593e223a5680c
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# npfimg-marker-feature-identification

## Summary

Identify multivariate chemo-/biomarker features from chromatography–MS data by treating the two-dimensional MS map (m/z vs retention time) as an image and combining image processing with machine learning via NPFimg, avoiding conventional peak picking that suffers false peak detections.

## When to use

Use when processing GC–MS or LC–MS chromatography–mass spectrometry data as a two-dimensional MS map (m/z vs retention time) to identify and visualize multivariate chemo-/biomarker features without conventional peak picking.

## When NOT to use

- When the goal is conventional peak detection/alignment via peak-picking tools such as XCMS and a peak table is the desired intermediate — NPFimg deliberately avoids the conventional peak picking process.
- When the input is not a two-dimensional MS map (m/z vs retention time) — e.g. non-chromatographic direct-infusion or NMR spectra — the image-based map approach does not apply.
- When data are single-channel 1D spectra without a retention-time axis, since NPFimg's discrimination and marker identification operate on 2D MS maps.

## Inputs

- Chromatography–MS raw data forming a two-dimensional MS map (m/z vs retention time), e.g. GC–MS data
- Two-dimensional MS maps from liquid chromatography–MS (potential applicability)

## Outputs

- Identified multivariate chemo-/biomarker features
- Visualization of marker features on the two-dimensional MS map

## How to apply

Load the chromatography–MS raw data and construct the two-dimensional MS map (m/z vs retention time) as an image rather than running conventional peak picking, which suffers from false peak detections. Run the NPFimg pipeline (open source at github.com/poomcj/NPFimg under the MIT license), which applies image processing combined with machine learning to the MS map to discriminate analytes and identify multivariate chemo-/biomarker features. Extract the identified marker features and visualize them on the two-dimensional MS map. The approach was validated on GC–MS case studies of aroma odor and human breath even at the parts per billion level, and comparison with XCMS showed NPFimg has lower error rates of signal acquisition and marker identification. While demonstrated on GC–MS, the pipeline is potentially applicable to LC–MS data processing in diverse metabolomics/chemometrics since both produce the same two-dimensional MS map format.

## Related tools

- **NPFimg** (Primary tool: combines image processing and machine learning on the 2D MS map (m/z vs retention time) to discriminate analytes, identify, and visualize multivariate chemo-/biomarker features without conventional peak picking.) — http://github.com/poomcj/NPFimg
- **XCMS** (Benchmark comparator: widely used peak-picking tool against which NPFimg's error rates of signal acquisition and marker identification were evaluated; XCMS represents the conventional approach NPFimg replaces.)

## Examples

```
git clone http://github.com/poomcj/NPFimg  # then run NPFimg per the full manual at pubs.acs.org/doi/full/10.1021/acs.analchem.1c03163
```

## Evaluation signals

- Marker feature identification succeeds on GC–MS case studies of aroma odor and human breath even at the parts per billion level — identified features should be detectable at ppb concentrations.
- Error rates of signal acquisition and marker identification are lower than those obtained with XCMS on the same chromatography–MS data.
- Signals are comprehensively characterized without false peak detections arising from conventional peak picking.
- Marker features are visualized on the two-dimensional MS map (m/z vs retention time) as discriminating analyte regions.
- Marker feature table can be produced from both GC–MS and (as applicability) liquid chromatography–MS inputs without modification of the core image-processing and machine-learning approach.

## Limitations

- The published study shows limited applications; NPFimg is described as 'potentially applicable' to diverse metabolomics/chemometrics with GC–MS and liquid chromatography–MS rather than exhaustively validated across them.
- Demonstrated feasibility is for GC–MS (aroma odor and human breath); LC–MS use is a stated potential, not a demonstrated case study.
- Quantitative validation was at the parts per billion level; performance below ppb was not shown.

## Evidence
