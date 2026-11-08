---
name: gcms-example-dataset-processing
description: Use when the input is raw two-dimensional chromatography–MS data (GC–MS
  or LC–MS, m/z vs retention time) and the goal is untargeted marker-feature identification
  while avoiding peak-picking false detections.
when_to_use_negative:
- Input is already a peak-picked feature table — NPFimg's core value is bypassing
  the conventional peak picking process, which suffers from false peak detections
- The analysis targets 1-D spectra or data that cannot be arranged as an m/z vs retention
  time 2-D map
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: NPFimg
  role: 'Primary pipeline: converts the 2-D MS map to image form and applies image
    processing plus machine learning to identify and visualize marker features without
    peak picking'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: Widely used comparator used to benchmark NPFimg's error rates of signal acquisition
    and marker identification
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:68122b50bf3c1ff9aa42c0791e7365543addd2d5f626e78daa8c737cd6d5fdd9
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# gcms-example-dataset-processing

## Summary

Processes a two-dimensional GC–MS map (m/z vs retention time) with NPFimg — an image-processing plus machine-learning pipeline — to identify and visualize multivariate chemo-/biomarker features without conventional peak picking, demonstrated on the package's human breath case study at parts-per-billion analyte levels.

## When to use

Use when the input is raw two-dimensional chromatography–MS data (GC–MS or LC–MS, m/z vs retention time) and the goal is untargeted marker-feature identification while avoiding peak-picking false detections.

## When NOT to use

- Input is already a peak-picked feature table — NPFimg's core value is bypassing the conventional peak picking process, which suffers from false peak detections
- The analysis targets 1-D spectra or data that cannot be arranged as an m/z vs retention time 2-D map

## Inputs

- Two-dimensional GC–MS map (m/z vs retention time), e.g. the human breath demonstration dataset shipped with NPFimg
- Image representation of the 2-D MS map as required by NPFimg

## Outputs

- Marker feature table (e.g., npfimg_breath_marker_features.csv) of identified multivariate chemo-/biomarker features
- Visualizations of the identified marker features on the 2-D MS map

## How to apply

Obtain NPFimg from github.com/poomcj/NPFimg (MIT license) and load the human breath GC–MS demonstration dataset shipped with the package. Convert the two-dimensional MS map (m/z vs retention time) into the image representation NPFimg requires for image processing. Run the NPFimg pipeline, which applies image processing and machine learning to discriminate analytes directly on the 2-D map rather than performing peak picking, producing marker features and their visualizations. Optionally benchmark against XCMS, since NPFimg reports lower error rates of signal acquisition and marker identification. Finally extract the marker feature table (e.g., save as npfimg_breath_marker_features.csv) and review it against the paper's reported breath case-study markers.

## Related tools

- **NPFimg** (Primary pipeline: converts the 2-D MS map to image form and applies image processing plus machine learning to identify and visualize marker features without peak picking) — http://github.com/poomcj/NPFimg
- **XCMS** (Widely used comparator used to benchmark NPFimg's error rates of signal acquisition and marker identification)

## Examples

```
git clone https://github.com/poomcj/NPFimg.git && python -m npfimg --input breath_gcms_demo --output npfimg_breath_marker_features.csv
```

## Evaluation signals

- The run completes and outputs a marker feature table consistent with the paper's human breath GC–MS demonstration at the parts-per-billion level
- Marker features are identified without invoking a conventional peak picking step (no peak table from tools like XCMS is required as input)
- Expert review of the output marker table against the reported aroma odor / human breath case-study markers
- If benchmarked, NPFimg shows lower error rates of signal acquisition and marker identification than XCMS
- Identified markers are accompanied by visualizations positioned on the m/z vs retention time map

## Limitations

- The study shows limited applications; NPFimg is 'potentially applicable' to diverse metabolomics/chemometrics with GC–MS and LC–MS but was demonstrated only on aroma odor and human breath GC–MS case studies
- Repository documentation is sparse — the README defers to the published manual ('For the other details, I will update soon')
- Conventional peak picking, which NPFimg replaces, is the standard workflow, so comparability requires an explicit XCMS comparison

## Evidence
