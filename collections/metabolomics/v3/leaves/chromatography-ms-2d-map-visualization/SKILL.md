---
name: chromatography-ms-2d-map-visualization
description: Use when input is a two-dimensional chromatography–MS map (m/z vs retention
  time) from GC–MS or LC–MS and you need to discriminate analytes and identify/visualize
  marker features without conventional peak picking.
when_to_use_negative:
- Input is already a peak-picked feature table rather than a raw two-dimensional m/z
  vs retention time map.
- Data are not chromatography–MS (e.g. spectra-only or non-chromatographic MS), since
  the skill requires an m/z vs retention time map representation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NPFimg
  role: 'Core pipeline: combines image processing and machine learning on the 2D MS
    map (m/z vs retention time) to discriminate analytes and identify and visualize
    marker features.'
  repo: github.com/poomcj/NPFimg
- name: XCMS
  role: Comparator tool used in the article as a benchmark for signal acquisition
    and marker identification error rates.
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:e4c3bd39f1ac5c97391d062efc31b9125bcf3230d71eed4b1eafe8ae8dafe4ed
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# chromatography-ms-2d-map-visualization

## Summary

Visualizes a two-dimensional chromatography–MS map (m/z vs retention time) as an image so that NPFimg can discriminate analytes and identify/visualize multivariate marker features without conventional peak picking. Supports GC–MS and LC–MS data pipelines.

## When to use

Use when input is a two-dimensional chromatography–MS map (m/z vs retention time) from GC–MS or LC–MS and you need to discriminate analytes and identify/visualize marker features without conventional peak picking.

## When NOT to use

- Input is already a peak-picked feature table rather than a raw two-dimensional m/z vs retention time map.
- Data are not chromatography–MS (e.g. spectra-only or non-chromatographic MS), since the skill requires an m/z vs retention time map representation.

## Inputs

- two-dimensional chromatography–MS map (m/z vs retention time) from GC–MS
- two-dimensional chromatography–MS map (m/z vs retention time) from LC–MS

## Outputs

- visualized marker features overlaid on the 2D MS map
- marker feature table with m/z, retention time, and marker classification

## How to apply

Convert the chromatography–MS acquisition into a two-dimensional MS map with m/z on one axis and retention time on the other — this image representation is the core input format of NPFimg for both GC–MS and liquid chromatography–MS data. Run the NPFimg pipeline, which combines image processing and machine learning, on this map to discriminate analytes and identify and visualize multivariate chemo-/biomarker features in place of the conventional peak picking process, which suffers from false peak detections. This visualization-based approach has been demonstrated at the parts per billion level for aroma odor and human breath GC–MS data, and compared against XCMS it shows lower error rates of signal acquisition and marker identification. Assemble the identified marker features into a marker feature table (feature identifier, m/z, retention time, marker classification) for downstream untargeted metabolomics of e.g. human breath.

## Related tools

- **NPFimg** (Core pipeline: combines image processing and machine learning on the 2D MS map (m/z vs retention time) to discriminate analytes and identify and visualize marker features.) — github.com/poomcj/NPFimg
- **XCMS** (Comparator tool used in the article as a benchmark for signal acquisition and marker identification error rates.)

## Examples

```
from npfimg import NPFimg; markers = NPFimg().fit_predict(ms_map)  # ms_map: 2D array (m/z vs retention time) from GC–MS/LC–MS
```

## Evaluation signals

- Output marker feature table must be parseable and contain m/z, retention time, and marker classification columns.
- Visualized marker features must be locatable on the 2D m/z vs retention time map.
- Marker identification should detect features at parts-per-billion concentration levels in the demonstrated GC–MS case studies.
- Signal acquisition and marker identification error rates should be lower than those obtained with XCMS on comparable data.
- No false peak detections attributable to a conventional peak picking step should be present in the output.

## Limitations

- Demonstrated primarily on GC–MS (aroma odor and human breath); LC–MS applicability is stated as potential and shown only in a limited set of applications.
- The article/README note the study shows limited applications, and the README states details will be updated — documentation is incomplete.
- Discriminates analytes on a 2D map basis; data without a retention-time axis are out of scope.

## Evidence

- [intro] NPFimg processes a two-dimensional MS map (m/z vs retention time) to discriminate analytes and identify and visualize marker features: "NPFimg processes a two-dimensional MS map (m/z vs retention time) to discriminate analytes and identify and visualize the marker features"
- [abstract] Avoids conventional peak picking: "Our approach allows us to comprehensively characterize the signals in MS data without the conventional peak picking process, which suffers from false peak detections."
- [abstract] Demonstrated at ppb level on GC–MS: "The feasibility of marker identification is successfully demonstrated in case studies of aroma odor and human breath on gas chromatography–mass spectrometry (GC–MS) even at the parts per billion"
- [abstract] Comparison with XCMS shows lower error rates: "Comparison with the widely used XCMS shows the excellent reliability of NPFimg, in that it has lower error rates of signal acquisition and marker identification."
