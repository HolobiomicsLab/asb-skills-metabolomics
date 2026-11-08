---
name: ms-marker-feature-visualization
description: Use when a GC–MS run has been processed with NPFimg (2D m/z–retention-time
  map, no conventional peak picking) and the identified multivariate marker features
  for analytes such as aroma odor or human breath metabolites need to be rendered
  on the original MS map for inspection or reporting.
when_to_use_negative:
- Input is already a curated peak/feature table from conventional peak picking rather
  than a 2D m/z–retention-time MS map processed by NPFimg — NPFimg's visualization
  is tied to its peak-picking-free map representation.
- Data are one-dimensional chromatograms or spectra with no m/z vs retention-time
  map, so there is no 2D MS map on which marker features can be visualized.
- The goal is de novo metabolite identification rather than visualizing marker features
  already identified by NPFimg's machine-learning stage.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: NPFimg
  role: 'Generates the marker-feature visualization: its pipeline discriminates analytes
    on the 2D m/z–retention-time MS map and identifies the multivariate chemo-/biomarker
    features that are rendered on the map'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: Baseline peak-picking-based tool whose signal-acquisition and marker-identification
    outputs can be visualized alongside NPFimg's to compare reliability
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_content_hash: sha256:2a79ecb07d519b5d60daf2aa470a3f20e6d62d3e8f601b13c805e8b84eed0011
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ms-marker-feature-visualization

## Summary

Visualize multivariate chemo-/biomarker features identified by NPFimg on a two-dimensional GC–MS map (m/z vs retention time), rendering the peak-picking-free marker calls produced by its image-processing and machine-learning stages as an interpretable map image.

## When to use

Use when a GC–MS run has been processed with NPFimg (2D m/z–retention-time map, no conventional peak picking) and the identified multivariate marker features for analytes such as aroma odor or human breath metabolites need to be rendered on the original MS map for inspection or reporting.

## When NOT to use

- Input is already a curated peak/feature table from conventional peak picking rather than a 2D m/z–retention-time MS map processed by NPFimg — NPFimg's visualization is tied to its peak-picking-free map representation.
- Data are one-dimensional chromatograms or spectra with no m/z vs retention-time map, so there is no 2D MS map on which marker features can be visualized.
- The goal is de novo metabolite identification rather than visualizing marker features already identified by NPFimg's machine-learning stage.

## Inputs

- Two-dimensional MS map (m/z vs retention time) from GC–MS data, e.g. the aroma odor or human breath example data shipped in the NPFimg repository
- Marker-feature set identified by NPFimg's image-processing + machine-learning pipeline (m/z and retention-time positions of multivariate chemo-/biomarker features)

## Outputs

- Marker-feature visualization image showing the identified chemo-/biomarker features on the two-dimensional MS map
- Optionally, comparison views against XCMS signal acquisitions/marker identifications to inspect error-rate differences

## How to apply

Load the two-dimensional MS map (m/z vs retention time) that NPFimg processed for the sample (e.g. the repository's aroma odor GC–MS example). Take the marker features output by NPFimg's image-processing plus machine-learning stages, which discriminate analytes without conventional peak picking and thus avoid the false peak detections that process suffers. Render these marker features back onto the 2D MS map exactly as NPFimg does, so the m/z and retention-time positions of chemo-/biomarker features are directly inspectable. If the same map was also run through XCMS, overlay or compare visualizations, since NPFimg reports lower error rates of signal acquisition and marker identification than XCMS — the visualization should be consistent with NPFimg's higher-reliability calls. Finally, save the rendered marker-feature map as the canonical output image of the workflow. The same rendering applies to human breath GC–MS data at parts-per-billion analyte levels.

## Related tools

- **NPFimg** (Generates the marker-feature visualization: its pipeline discriminates analytes on the 2D m/z–retention-time MS map and identifies the multivariate chemo-/biomarker features that are rendered on the map) — http://github.com/poomcj/NPFimg
- **XCMS** (Baseline peak-picking-based tool whose signal-acquisition and marker-identification outputs can be visualized alongside NPFimg's to compare reliability)

## Examples

```
python npfimg.py --input aroma_odor_gcms_map --visualize-markers --output marker_feature_map.png
```

## Evaluation signals

- The output visualization image is produced and displays marker features at concrete m/z and retention-time positions on the original 2D MS map.
- The marker features shown correspond to the analyte set from the aroma odor or human breath GC–MS case study, including features at parts-per-billion levels.
- Visualized features match NPFimg's marker identifications, which exhibit lower signal-acquisition and marker-identification error rates than XCMS on the same map.
- No visualization artifacts traceable to conventional peak picking (false peak detections) appear, since NPFimg bypasses that process.
- The saved visualization is consistent with the marker-feature pattern reported in the paper's aroma odor and human breath case-study figures.

## Limitations

- NPFimg (and hence the visualization) was demonstrated on GC–MS data (aroma odor, human breath) with limited scope; applicability to LC–MS data processing in diverse metabolomics/chemometrics is described as potential rather than shown in the study.
- Conventional peak picking, which the visualization's underlying pipeline avoids, suffers from false peak detections — visualization of features from such peak tables would inherit those errors.
- Marker identification reliability was validated down to the parts-per-billion level; visualizing features below that demonstrated concentration range is not supported by the reported evidence.

## Evidence
