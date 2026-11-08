---
name: breath-metabolomics-untargeted-analysis
description: Use when the input is two-dimensional GC–MS (or LC–MS) chromatography
  data — e.g. human breath or aroma-odor samples — and the goal is untargeted marker
  identification without peak picking, especially at ppb-level concentrations.
when_to_use_negative:
- The input is already a peak-picked or extracted feature table — NPFimg is designed
  to operate directly on the two-dimensional MS map without conventional peak picking
- The analysis requires the conventional peak-picking workflow (e.g. XCMS-style peak
  detection) rather than image-based marker identification
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NPFimg
  role: 'Primary pipeline: converts the 2D MS map to an image, applies image processing
    and machine learning to identify and visualize marker features without peak picking'
  repo: github.com/poomcj/NPFimg
- name: XCMS
  role: Baseline comparator used in the article to benchmark NPFimg's error rates
    of signal acquisition and marker identification
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:9ef09a94bf8e3f5ebe1e7bd57f4e1b402b5447fbfbab6bb0451ffba193f4d09a
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# breath-metabolomics-untargeted-analysis

## Summary

Applies NPFimg, an image-processing and machine-learning method, to two-dimensional GC–MS maps (m/z vs retention time) of human breath to identify multivariate chemo-/biomarker features without conventional peak picking, even at parts-per-billion analyte levels.

## When to use

Use when the input is two-dimensional GC–MS (or LC–MS) chromatography data — e.g. human breath or aroma-odor samples — and the goal is untargeted marker identification without peak picking, especially at ppb-level concentrations.

## When NOT to use

- The input is already a peak-picked or extracted feature table — NPFimg is designed to operate directly on the two-dimensional MS map without conventional peak picking
- The analysis requires the conventional peak-picking workflow (e.g. XCMS-style peak detection) rather than image-based marker identification

## Inputs

- Two-dimensional GC–MS map (m/z vs retention time), e.g. human breath GC–MS data converted to the image representation NPFimg requires
- NPFimg demonstration case-study dataset (aroma odor / human breath GC–MS) shipped with the package

## Outputs

- Marker feature table (e.g. npfimg_breath_marker_features.csv) of multivariate chemo-/biomarker features
- Visualizations of the identified marker features overlaid on the 2D MS map

## How to apply

Install NPFimg from github.com/poomcj/NPFimg (MIT license) and load the human breath GC–MS demonstration dataset shipped with the package. Convert the two-dimensional MS map (m/z vs retention time) into the image representation NPFimg requires. Run the NPFimg pipeline, which combines image processing and machine learning to discriminate analytes and identify marker features directly from the map, bypassing conventional peak picking, which suffers from false peak detections. Extract the identified marker features and their visualizations from the output. As a benchmark, NPFimg was compared with XCMS and showed lower error rates of signal acquisition and marker identification, so results should be plausible relative to that baseline.

## Related tools

- **NPFimg** (Primary pipeline: converts the 2D MS map to an image, applies image processing and machine learning to identify and visualize marker features without peak picking) — github.com/poomcj/NPFimg
- **XCMS** (Baseline comparator used in the article to benchmark NPFimg's error rates of signal acquisition and marker identification)

## Evaluation signals

- The pipeline run completes and outputs marker features consistent with the paper's demonstrated human breath case study
- Marker features are identified from the m/z vs retention time map without any intermediate conventional peak-picking step
- The assembled marker feature table (npfimg_breath_marker_features.csv) matches markers reported for the breath case at the parts-per-billion level
- Expert review of the output table against the paper's reported breath case study markers confirms correspondence
- Error rates of signal acquisition and marker identification are at or below those observed for XCMS in the article's comparison

## Limitations

- The study demonstrates only limited applications (aroma odor and human breath GC–MS); broader applicability to diverse metabolomics/chemometrics with GC–MS and LC–MS is stated as potential, not proven
- Marker identification was demonstrated even at the parts per billion level, but sensitivity at lower concentrations is not established in the provided context
- The repository README notes documentation is incomplete ('For the other details, I will update soon')

## Evidence

- [abstract] NPFimg processes a two-dimensional MS map (m/z vs retention time) to discriminate analytes and identify and visualize marker features: "NPFimg processes a two-dimensional MS map ( m / z vs retention time) to discriminate analytes and identify and visualize the marker features."
- [intro] Marker identification demonstrated for human breath GC–MS even at parts per billion: "The feasibility of marker identification is successfully demonstrated in case studies of aroma odor and human breath on gas chromatography–mass spectrometry (GC–MS) even at the parts per billion level"
- [abstract] NPFimg has lower error rates of signal acquisition and marker identification compared with XCMS: "Comparison with the widely used XCMS shows the excellent reliability of NPFimg, in that it has lower error rates of signal acquisition and marker identification."
- [readme] NPFimg avoids conventional peak picking, which suffers from false peak detections: "Our approach allows us to comprehensively characterize the signals in MS data without the conventional peak picking process, which suffers from false peak detections."
- [abstract] NPFimg available open source under MIT license on GitHub: "NPFimg is available as open source on GitHub ( http://github.com/poomcj/NPFimg ) under the MIT license."
