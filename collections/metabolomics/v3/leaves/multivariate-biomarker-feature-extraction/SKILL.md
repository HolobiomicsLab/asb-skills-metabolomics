---
name: multivariate-biomarker-feature-extraction
description: Use when input is two-dimensional MS map data (m/z vs retention time)
  from GC–MS or LC–MS and the goal is to identify and visualize multivariate marker
  features without conventional peak picking, e.g. untargeted metabolomics of aroma
  odor or human breath.
when_to_use_negative:
- Input is already a peak-picked feature table — NPFimg's value is working directly
  on the 2D MS map image, bypassing peak picking.
- The task requires a long-established, widely supported peak picking workflow — the
  article itself benchmarks against 'the widely used XCMS', implying XCMS is the conventional
  alternative.
- The data is not a two-dimensional m/z vs retention-time MS map (e.g. no chromatographic
  dimension or non-MS modality) — NPFimg is defined for chromatography–MS data.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NPFimg
  role: 'Core pipeline: combines image processing and machine learning on the 2D MS
    map to discriminate analytes and identify/visualize marker features without peak
    picking'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: Conventional comparator pipeline used to benchmark NPFimg's error rates of
    signal acquisition and marker identification
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:e1f14e65a725a2627aff4b9a2627a9586973c548834c5cfac0ed798d745cf7a3
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# multivariate-biomarker-feature-extraction

## Summary

NPFimg automatically identifies multivariate chemo-/biomarker features of analytes in chromatography–mass spectrometry data by combining image processing and machine learning on two-dimensional MS maps (m/z vs retention time), bypassing the conventional peak picking process that suffers from false peak detections. Validated on GC–MS aroma odor and human breath data (down to the parts per billion level) and shown to have lower signal-acquisition and marker-identification error rates than XCMS.

## When to use

Use when input is two-dimensional MS map data (m/z vs retention time) from GC–MS or LC–MS and the goal is to identify and visualize multivariate marker features without conventional peak picking, e.g. untargeted metabolomics of aroma odor or human breath.

## When NOT to use

- Input is already a peak-picked feature table — NPFimg's value is working directly on the 2D MS map image, bypassing peak picking.
- The task requires a long-established, widely supported peak picking workflow — the article itself benchmarks against 'the widely used XCMS', implying XCMS is the conventional alternative.
- The data is not a two-dimensional m/z vs retention-time MS map (e.g. no chromatographic dimension or non-MS modality) — NPFimg is defined for chromatography–MS data.

## Inputs

- Two-dimensional MS map (m/z vs retention time) from chromatography–MS data, e.g. GC–MS of aroma odor or human breath
- LC–MS two-dimensional data (m/z vs retention time), per the article's statement of potential applicability to liquid chromatography–MS

## Outputs

- Identified and visualized multivariate chemo-/biomarker features of analytes (marker feature table with feature coordinates on the m/z vs retention time map)
- Signal characterization of the MS data without peak picking
- Error-rate comparison metrics (signal acquisition and marker identification) when benchmarked against XCMS

## How to apply

Load the two-dimensional MS map (m/z vs retention time) — the shared data format of GC–MS and LC–MS — and feed it directly to NPFimg's image-processing plus machine-learning pipeline to discriminate analytes; do not run a separate peak picking step, since NPFimg comprehensively characterizes signals without it and conventional peak picking suffers from false peak detections. The pipeline outputs identified marker features with coordinates for visualization. When validation against a conventional pipeline is needed, run XCMS on the same data and compare error rates of signal acquisition and marker identification, where NPFimg was reported as the lower-error method. For sensitivity, NPFimg demonstrated marker identification on GC–MS aroma odor and human breath data at the parts per billion level.

## Related tools

- **NPFimg** (Core pipeline: combines image processing and machine learning on the 2D MS map to discriminate analytes and identify/visualize marker features without peak picking) — http://github.com/poomcj/NPFimg
- **XCMS** (Conventional comparator pipeline used to benchmark NPFimg's error rates of signal acquisition and marker identification)

## Examples

```
git clone http://github.com/poomcj/NPFimg; python npfimg.py --input ms_map.mzvsrt --output marker_features.csv
```

## Evaluation signals

- Output contains identified marker features with coordinates on the m/z vs retention-time map that can be visualized (marker feature table is populated, not empty, for a case with known analytes).
- Signal acquisition error rate and marker identification error rate from NPFimg are lower than the corresponding XCMS values when both are run on the same dataset (the paper's reported benchmark direction).
- No conventional peak picking was performed as a prerequisite — the pipeline ingested the raw 2D MS map directly (absence of a separate peak-picking step/output artifact).
- Markers are detectable at the parts per billion level on GC–MS aroma odor or human breath data, matching the paper's demonstrated sensitivity range.
- Downstream metabolomics outputs (e.g. untargeted human breath metabolomics feature sets) are produced from the marker table without manual peak curation.

## Limitations

- Demonstrated case studies are limited to aroma odor and human breath on GC–MS; the article frames broader coverage (including LC–MS) as 'potentially applicable', i.e. not experimentally shown.
- The GitHub repository README states documentation is incomplete: 'For the other details, I will update soon' — users may need to consult the ACS full-text manual script for usage details.
- Conventional peak picking is not used, so features unavailable in image-based representation may not be recoverable; the paper claims the approach 'comprehensively characterize[s] the signals', but false-peak-related edge cases of peak picking are by design outside its scope.

## Evidence
