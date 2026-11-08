---
name: machine-learning-analyte-discrimination
description: Use when GC–MS or LC–MS chromatography–MS data must be mined for multivariate
  chemo-/biomarker marker features at very low (parts-per-billion) analyte concentrations
  and conventional peak picking is undesirable, e.g., untargeted metabolomics of aroma
  odor or human breath samples.
when_to_use_negative:
- Input is already a processed peak/feature table, since NPFimg's value is characterizing
  signals in the raw two-dimensional MS map without peak picking
- The data lack a two-dimensional (m/z vs retention time) chromatography–MS structure,
  which NPFimg's image representation requires
- Only individual-peak detection for spectral library matching is needed rather than
  multivariate marker feature discrimination across samples
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NPFimg
  role: 'Primary tool: combines image processing and machine learning on the 2D MS
    map (m/z vs retention time) to discriminate analytes and identify/visualize marker
    features without peak picking'
  repo: github.com/poomcj/NPFimg
- name: XCMS
  role: Benchmark comparator used to assess reliability, since NPFimg shows lower
    error rates of signal acquisition and marker identification
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_content_hash: sha256:bcdebeabc08cdb0e1131285205ccabf9f73bdefad0662f705a5fc210b9e6b5e9
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# machine-learning-analyte-discrimination

## Summary

Apply NPFimg, an image-processing plus machine-learning method, to two-dimensional chromatography–MS maps (m/z vs retention time) to discriminate analytes and identify multivariate chemo-/biomarker marker features without conventional peak picking, which suffers from false peak detections.

## When to use

Use when GC–MS or LC–MS chromatography–MS data must be mined for multivariate chemo-/biomarker marker features at very low (parts-per-billion) analyte concentrations and conventional peak picking is undesirable, e.g., untargeted metabolomics of aroma odor or human breath samples.

## When NOT to use

- Input is already a processed peak/feature table, since NPFimg's value is characterizing signals in the raw two-dimensional MS map without peak picking
- The data lack a two-dimensional (m/z vs retention time) chromatography–MS structure, which NPFimg's image representation requires
- Only individual-peak detection for spectral library matching is needed rather than multivariate marker feature discrimination across samples

## Inputs

- GC–MS or liquid chromatography–MS chromatography–MS raw data (e.g., the aroma odor GC–MS example dataset shipped with the NPFimg repository)
- Two-dimensional MS map (m/z vs retention time) ready for conversion to NPFimg's image representation
- Human breath GC–MS demonstration dataset shipped with NPFimg (untargeted metabolomics case)

## Outputs

- Marker feature table of identified multivariate chemo-/biomarker features (e.g., marker_feature_table.csv for aroma odor, npfimg_breath_marker_features.csv for human breath)
- Visualizations of identified marker features positioned on the 2D MS map (m/z vs retention time)

## How to apply

Clone or install the open-source NPFimg repository (github.com/poomcj/NPFimg, MIT license) and load its shipped example datasets (aroma odor GC–MS or human breath GC–MS case studies). Convert the two-dimensional MS map (m/z vs retention time) into the image representation NPFimg requires, since the method treats the map as an image. Run the NPFimg pipeline, which combines image processing and machine learning to discriminate analytes and identify and visualize marker features directly on the 2D MS map, bypassing the conventional peak picking process that suffers from false peak detections. Extract the identified marker features (with their m/z–retention-time positions and visualizations) into an output table such as marker_feature_table.csv or npfimg_breath_marker_features.csv. Benchmark reliability against XCMS, which the paper reports has higher error rates of signal acquisition and marker identification than NPFimg.

## Related tools

- **NPFimg** (Primary tool: combines image processing and machine learning on the 2D MS map (m/z vs retention time) to discriminate analytes and identify/visualize marker features without peak picking) — github.com/poomcj/NPFimg
- **XCMS** (Benchmark comparator used to assess reliability, since NPFimg shows lower error rates of signal acquisition and marker identification)

## Evaluation signals

- The NPFimg pipeline run completes and outputs a marker feature table with entries for the processed 2D MS maps
- Identified marker features and their m/z–retention-time positions qualitatively match the paper's reported aroma odor and human breath case study markers (expert_review)
- Marker features are recovered even for analytes present at the parts-per-billion level, as demonstrated in both case studies
- Error rates of signal acquisition and marker identification compare favorably against an XCMS run on the same data
- Marker features can be visualized on the 2D MS map as in the paper's case study figures

## Limitations

- The study demonstrates limited applications — marker identification for aroma odor and human breath GC–MS case studies — though NPFimg is potentially applicable to diverse metabolomics/chemometrics using GC–MS and liquid chromatography–MS
- The repository README states 'For the other details, I will update soon', indicating sparse documentation; the full manual script is only in the published ACS paper

## Evidence
