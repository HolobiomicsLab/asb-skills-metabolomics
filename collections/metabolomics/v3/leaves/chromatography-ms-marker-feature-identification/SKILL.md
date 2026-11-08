---
name: chromatography-ms-marker-feature-identification
description: Use when you have raw two-dimensional chromatography–MS maps (GC–MS or
  LC–MS, e.g. m/z vs retention time) and need untargeted marker feature identification
  without prior peak picking.
when_to_use_negative:
- Your analysis requires conventional peak picking output or a curated feature table
  as the starting point — NPFimg instead works directly on the raw two-dimensional
  MS map to avoid false peak detections
- The data are not chromatography–MS maps (e.g. direct-injection MS or non-chromatographic
  spectra); NPFimg's demonstrated scope is GC–MS and liquid chromatography–MS data
  processing
- A targeted quantification pipeline with validated peak integration is required rather
  than untargeted marker feature discovery
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NPFimg
  role: 'Primary method: image processing + machine learning pipeline that processes
    the two-dimensional MS map (m/z vs retention time) to discriminate analytes and
    identify/visualize marker features without conventional peak picking'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: 'Benchmark comparator: widely used peak-picking tool whose error rates of
    signal acquisition and marker identification are lower for NPFimg, used to assess
    reliability'
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:9138b9ff4df546e017a27b522e7091e20dd36ccf2680e0e75bb421ddf8ec28ef
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# chromatography-ms-marker-feature-identification

## Summary

Apply NPFimg to identify multivariate chemo-/biomarker features in chromatography–MS data by combining image processing and machine learning on the two-dimensional MS map (m/z vs retention time), avoiding conventional peak picking and its false peak detections. Demonstrated on human breath and aroma odor GC–MS data down to the parts per billion level.

## When to use

Use when you have raw two-dimensional chromatography–MS maps (GC–MS or LC–MS, e.g. m/z vs retention time) and need untargeted marker feature identification without prior peak picking.

## When NOT to use

- Your analysis requires conventional peak picking output or a curated feature table as the starting point — NPFimg instead works directly on the raw two-dimensional MS map to avoid false peak detections
- The data are not chromatography–MS maps (e.g. direct-injection MS or non-chromatographic spectra); NPFimg's demonstrated scope is GC–MS and liquid chromatography–MS data processing
- A targeted quantification pipeline with validated peak integration is required rather than untargeted marker feature discovery

## Inputs

- Two-dimensional chromatography–MS map (m/z vs retention time), e.g. raw human breath GC–MS data or the NPFimg demonstration case study dataset
- Image representation of the MS map as required by NPFimg

## Outputs

- Marker feature table (e.g. npfimg_breath_marker_features.csv) with multivariate chemo-/biomarker features
- Visualizations of the identified marker features on the two-dimensional MS map

## How to apply

Install NPFimg from github.com/poomcj/NPFimg (MIT license). Load the chromatography–MS dataset (for the human breath case, use the GC–MS example data shipped with the package) and convert the two-dimensional MS map (m/z vs retention time) into the image representation NPFimg requires. Run the NPFimg pipeline, which combines image processing and machine learning to discriminate analytes directly on the map, bypassing conventional peak picking, and produce marker features. Extract the identified marker features and their visualizations from the output — for the human breath case study this succeeds even at the parts per billion level — and assemble them into a marker feature table. NPFimg has been benchmarked against XCMS and shows lower error rates of signal acquisition and marker identification, which serves as a reference point for validating results.

## Related tools

- **NPFimg** (Primary method: image processing + machine learning pipeline that processes the two-dimensional MS map (m/z vs retention time) to discriminate analytes and identify/visualize marker features without conventional peak picking) — http://github.com/poomcj/NPFimg
- **XCMS** (Benchmark comparator: widely used peak-picking tool whose error rates of signal acquisition and marker identification are lower for NPFimg, used to assess reliability)

## Evaluation signals

- Pipeline run completes and outputs a marker feature table with the identified multivariate chemo-/biomarker features and visualizations
- For the human breath GC–MS case, marker features are detected down to parts per billion concentrations, consistent with the published case study
- Marker identification error rate is comparable to or lower than the XCMS benchmark (NPFimg reported lower error rates of signal acquisition and marker identification)
- No conventional peak picking step was applied — outputs derive directly from the two-dimensional MS map image processing
- Marker features and visualizations align with the paper's demonstrated aroma odor and human breath case study markers (expert review)

## Limitations

- The study shows limited applications; the authors state NPFimg is only potentially applicable to diverse metabolomics/chemometrics using GC–MS and liquid chromatography–MS, so other data modalities are untested
- Conventional peak picking, which NPFimg replaces, suffers from false peak detections — comparisons to peak-picked feature lists may not be directly comparable
- Repository documentation is sparse: the README defers to the full manual in the journal article and states 'For the other details, I will update soon', so parameter guidance must come from the paper

## Evidence
