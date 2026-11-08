---
name: gcms-data-processing
description: Use when raw GC–MS (or LC–MS) chromatogram data must be converted into
  marker-feature identifications for metabolomics/chemometrics, and conventional peak
  picking (e.g. XCMS) is producing false peak detections or unreliable signal acquisition.
when_to_use_negative:
- The input is already a processed feature table or peak list rather than a two-dimensional
  MS map — NPFimg operates on the raw m/z vs retention-time map, not on pre-picked
  peaks.
- The analysis requires a conventional peak-picking workflow whose outputs (retention-time/peak
  tables) downstream tools depend on — NPFimg deliberately bypasses peak picking,
  so pipelines expecting peak tables would not integrate directly.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: NPFimg
  role: 'Primary tool: automatically identifies multivariate chemo-/biomarker features
    of analytes in chromatography–MS data by combining image processing and machine
    learning on the two-dimensional MS map, without conventional peak picking.'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: 'Benchmark comparator: performs conventional peak detection and marker identification
    on the same GC–MS datasets to quantify error rates of signal acquisition and marker
    identification against NPFimg.'
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:130ca2e61cf14c17a9bc9f20efc149b3710fe37c223796b8188ee43212301d4f
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gcms-data-processing

## Summary

Process two-dimensional GC–MS maps (m/z vs retention time) with NPFimg, an image-processing + machine-learning pipeline that identifies and visualizes multivariate chemo-/biomarker features without conventional peak picking, which is prone to false peak detections.

## When to use

Use when raw GC–MS (or LC–MS) chromatogram data must be converted into marker-feature identifications for metabolomics/chemometrics, and conventional peak picking (e.g. XCMS) is producing false peak detections or unreliable signal acquisition.

## When NOT to use

- The input is already a processed feature table or peak list rather than a two-dimensional MS map — NPFimg operates on the raw m/z vs retention-time map, not on pre-picked peaks.
- The analysis requires a conventional peak-picking workflow whose outputs (retention-time/peak tables) downstream tools depend on — NPFimg deliberately bypasses peak picking, so pipelines expecting peak tables would not integrate directly.

## Inputs

- Two-dimensional chromatography–MS maps (m/z vs retention time) from GC–MS experiments, e.g. aroma-odor and human-breath GC–MS datasets
- Raw chromatogram data from liquid chromatography–MS (potential applicability)

## Outputs

- Identified and visualized multivariate chemo-/biomarker features (marker-feature maps over the m/z vs retention-time plane)
- Signal-acquisition results and marker-identification results with associated error rates when benchmarked against XCMS

## How to apply

Load the raw chromatography–MS data as a two-dimensional MS map (m/z vs retention time) and run NPFimg, which combines image processing with machine learning to discriminate analytes directly on the map rather than via peak picking; this avoids false peak detections inherent to conventional peak-picking pipelines. In the demonstrated case studies, NPFimg was applied to aroma-odor and human-breath GC–MS data down to the parts-per-billion level, identifying and visualizing marker features such as biomarker-feature groups of analytes. As a benchmark, run XCMS on the same datasets for peak detection and marker identification, then count erroneous signal-acquisition events and erroneous marker identifications for each method and compute error rates as erroneous events divided by the total evaluated signals/markers; NPFimg is expected to yield lower error rates on both metrics. The tool is open source (MIT license) at github.com/poomcj/NPFimg and is potentially applicable to diverse metabolomics/chemometrics using GC–MS and liquid chromatography–MS.

## Related tools

- **NPFimg** (Primary tool: automatically identifies multivariate chemo-/biomarker features of analytes in chromatography–MS data by combining image processing and machine learning on the two-dimensional MS map, without conventional peak picking.) — http://github.com/poomcj/NPFimg
- **XCMS** (Benchmark comparator: performs conventional peak detection and marker identification on the same GC–MS datasets to quantify error rates of signal acquisition and marker identification against NPFimg.)

## Examples

```
git clone http://github.com/poomcj/NPFimg && python npfimg.py --input aroma_gcms_data --output marker_features
```

## Evaluation signals

- The output marker-feature maps should be visualized over the m/z vs retention-time plane and correspond to multivariate features of analytes, not individual picked peaks.
- In the head-to-head benchmark on the same GC–MS datasets, NPFimg must show lower error rates of signal acquisition and marker identification than XCMS.
- Marker identification should succeed even at parts-per-billion concentrations in the aroma-odor and human-breath GC–MS case studies.
- The NPFimg run must not rely on a conventional peak-picking step; absence of false peak detections characteristic of peak picking is expected.
- The tool distribution is the open-source repository github.com/poomcj/NPFimg under the MIT license; running the code from that repo is a sanity check.

## Limitations

- The study demonstrates only limited applications (aroma-odor and human-breath GC–MS), though the authors state NPFimg is potentially applicable to diverse metabolomics/chemometrics using GC–MS and liquid chromatography–MS.
- Conventional peak-picking approaches (e.g. XCMS) suffer from false peak detections, which is the failure mode NPFimg is designed to avoid; direct comparison requires running both methods on identical data.
- The README notes that additional documentation is pending ('For the other details, I will update soon'), so users may need to consult the full manual script in the published article.

## Evidence
