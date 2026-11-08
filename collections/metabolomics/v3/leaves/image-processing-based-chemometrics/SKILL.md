---
name: image-processing-based-chemometrics
description: Use when raw GC-MS or LC-MS two-dimensional MS maps (m/z vs retention
  time) must be characterized comprehensively without peak picking, and multivariate
  marker features need to be identified and visualized, even at parts-per-billion
  analyte concentrations.
when_to_use_negative:
- Input is already a curated peak/feature table where peak picking has been performed
  and validated - NPFimg's value is bypassing the peak picking process, which suffers
  from false peak detections.
- The goal is a targeted analysis of a small set of known analytes rather than untargeted
  identification of multivariate marker features across a 2D MS map.
- Only a 1D spectrum is available with no two-dimensional m/z vs retention time map,
  since the method processes 2D MS maps.
tools:
- name: NPFimg
  role: 'Primary tool: combines image processing and machine learning on the 2D MS
    map to discriminate analytes and identify/visualize marker features without peak
    picking.'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: 'Benchmark comparator: run alongside NPFimg on the same data to quantify signal
    acquisition and marker identification error rates, demonstrating NPFimg''s lower
    error rates.'
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:bb58aad10cc2ed65038646cd7ebad2009c30bdd1a004b977e4a5e7620c39ac36
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# image-processing-based-chemometrics

## Summary

Identifies multivariate chemo-/biomarker features from chromatography-MS data by converting the two-dimensional MS map (m/z vs retention time) into an image and applying image processing plus machine learning, avoiding the conventional peak picking process that suffers from false peak detections. Implemented in NPFimg and demonstrated on GC-MS aroma odor and human breath data down to the parts per billion level.

## When to use

Use when raw GC-MS or LC-MS two-dimensional MS maps (m/z vs retention time) must be characterized comprehensively without peak picking, and multivariate marker features need to be identified and visualized, even at parts-per-billion analyte concentrations.

## When NOT to use

- Input is already a curated peak/feature table where peak picking has been performed and validated - NPFimg's value is bypassing the peak picking process, which suffers from false peak detections.
- The goal is a targeted analysis of a small set of known analytes rather than untargeted identification of multivariate marker features across a 2D MS map.
- Only a 1D spectrum is available with no two-dimensional m/z vs retention time map, since the method processes 2D MS maps.

## Inputs

- two-dimensional MS map (m/z vs retention time) from gas chromatography-mass spectrometry (GC-MS) data
- chromatography-mass spectrometry (MS) data such as aroma odor or human breath GC-MS measurements at parts-per-billion analyte levels
- liquid chromatography-MS data (potential applicability)

## Outputs

- identified multivariate chemo-/biomarker features of analytes
- visualizations of marker features on the two-dimensional MS map
- signal acquisition and marker identification error rates (for reliability comparison against XCMS)

## How to apply

Obtain NPFimg from the GitHub repository (github.com/poomcj/NPFimg, MIT license) and load the chromatography-MS data to construct the two-dimensional MS map of m/z versus retention time. Treat this map as an image and run the NPFimg image-processing and machine-learning pipeline to discriminate analytes and identify multivariate chemo-/biomarker features, deliberately skipping the conventional peak picking step because it suffers from false peak detections. Visualize the identified marker features on the 2D MS map so signals are comprehensively characterized. When reliability must be quantified, run the widely used XCMS on the same data and compare error rates of signal acquisition and marker identification, expecting NPFimg to show lower error rates. The approach is applicable to diverse metabolomics/chemometrics using GC-MS and liquid chromatography-MS.

## Related tools

- **NPFimg** (Primary tool: combines image processing and machine learning on the 2D MS map to discriminate analytes and identify/visualize marker features without peak picking.) — http://github.com/poomcj/NPFimg
- **XCMS** (Benchmark comparator: run alongside NPFimg on the same data to quantify signal acquisition and marker identification error rates, demonstrating NPFimg's lower error rates.)

## Examples

```
git clone http://github.com/poomcj/NPFimg  # then run the NPFimg pipeline on the aroma odor GC-MS 2D MS map per the manual script at https://pubs.acs.org/doi/full/10.1021/acs.analchem.1c03163
```

## Evaluation signals

- The pipeline runs without invoking any conventional peak picking process; check that marker identification proceeds directly on the 2D MS map image.
- Marker features are identified and visualized on the m/z vs retention time map for case-study data (aroma odor and human breath GC-MS) at the parts per billion level.
- Compared with XCMS on the same dataset, NPFimg yields lower error rates of signal acquisition and marker identification.
- Output visualizations show the discriminating multivariate features that separate analyte groups in the 2D MS map.

## Limitations

- The conventional peak picking process it replaces suffers from false peak detections, but the study shows limited applications overall; NPFimg is only 'potentially applicable' to diverse metabolomics/chemometrics using GC-MS and liquid chromatography-MS.
- Demonstrated feasibility covers aroma odor and human breath GC-MS case studies; applicability beyond these case studies is stated as potential rather than validated.
- The repository README notes documentation is incomplete ('For the other details, I will update soon'), so full parameter-level details must be taken from the paper's manual script.

## Evidence
