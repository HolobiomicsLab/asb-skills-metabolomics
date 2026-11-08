---
name: ms-signal-characterization-without-peak-picking
description: Use when raw 2D chromatography–MS data (GC–MS or LC–MS maps of m/z vs
  retention time) must be comprehensively characterized and marker features identified
  and visualized, and conventional peak picking is either unavailable, unreliable
  (prone to false peak detections), or undesired — e.
when_to_use_negative:
- Input is already a peak-picked feature table or a processed analyte list — this
  skill specifically operates on the raw two-dimensional MS map without peak picking,
  so applying it to already-reduced feature data is inappropriate.
- The analysis goal is conventional peak picking / peak detection itself rather than
  marker-feature identification from image-domain signal characterization — this skill's
  rationale explicitly replaces the peak-picking process.
- The target is not a chromatography–MS map (e.g., single-dimension MS-only spectra
  without a retention-time dimension, or non-chromatographic spectroscopy) — NPFimg
  is designed for GC–MS and LC–MS data with m/z vs retention time structure.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NPFimg
  role: Primary tool implementing the image-processing-plus-machine-learning pipeline
    that processes the two-dimensional MS map to discriminate analytes, identify,
    and visualize multivariate chemo-/biomarker features without conventional peak
    picking.
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: 'Benchmark/comparison tool used to assess NPFimg''s reliability: NPFimg should
    demonstrate lower error rates of signal acquisition and marker identification
    relative to XCMS.'
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:41fddc96d764bcbc3d82de736f3ed00992916fd2897923c2f30b8f8f3c2b3239
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# ms-signal-characterization-without-peak-picking

## Summary

Skill for automatically identifying and visualizing multivariate chemo-/biomarker features of analytes in chromatography–mass spectrometry (GC–MS or LC–MS) data by combining image processing and machine learning on a two-dimensional MS map (m/z vs retention time), explicitly bypassing the conventional peak-picking process that suffers from false peak detections. Implemented by the open-source tool NPFimg.

## When to use

Use when raw 2D chromatography–MS data (GC–MS or LC–MS maps of m/z vs retention time) must be comprehensively characterized and marker features identified and visualized, and conventional peak picking is either unavailable, unreliable (prone to false peak detections), or undesired — e.g., untargeted metabolomics/chemometrics of aroma odor or human breath at parts-per-billion analyte levels.

## When NOT to use

- Input is already a peak-picked feature table or a processed analyte list — this skill specifically operates on the raw two-dimensional MS map without peak picking, so applying it to already-reduced feature data is inappropriate.
- The analysis goal is conventional peak picking / peak detection itself rather than marker-feature identification from image-domain signal characterization — this skill's rationale explicitly replaces the peak-picking process.
- The target is not a chromatography–MS map (e.g., single-dimension MS-only spectra without a retention-time dimension, or non-chromatographic spectroscopy) — NPFimg is designed for GC–MS and LC–MS data with m/z vs retention time structure.

## Inputs

- Raw two-dimensional chromatography–MS map (m/z vs retention time intensity image), e.g., GC–MS data from aroma odor or human breath samples; LC–MS data also supported

## Outputs

- Discriminated analyte identities from the MS map
- Identified multivariate chemo-/biomarker features
- Visualizations of the identified marker features overlaid on the two-dimensional MS map
- Signal-acquisition and marker-identification error-rate metrics comparable to XCMS

## How to apply

Load the raw two-dimensional MS map (m/z vs retention time) from chromatography–mass spectrometry data (GC–MS demonstrated for aroma odor and human breath; LC–MS also supported) as an image to be processed by NPFimg. Apply NPFimg's image-processing stage to comprehensively characterize the signals in the MS map in the image domain, deliberately skipping the conventional peak-picking process because it suffers from false peak detections. Then apply NPFimg's machine-learning stage to discriminate analytes and automatically identify multivariate chemo-/biomarker features. Render the identified marker features as visualizations overlaid on the two-dimensional MS map. Compare against XCMS as a reliability benchmark: NPFimg is expected to show lower error rates of both signal acquisition and marker identification. Rationale: treating the MS map as a 2D image allows signal characterization without discrete peak calls, avoiding false peak detections while retaining sensitivity down to parts-per-billion analyte levels demonstrated in the GC–MS case studies.

## Related tools

- **NPFimg** (Primary tool implementing the image-processing-plus-machine-learning pipeline that processes the two-dimensional MS map to discriminate analytes, identify, and visualize multivariate chemo-/biomarker features without conventional peak picking.) — http://github.com/poomcj/NPFimg
- **XCMS** (Benchmark/comparison tool used to assess NPFimg's reliability: NPFimg should demonstrate lower error rates of signal acquisition and marker identification relative to XCMS.)

## Examples

```
git clone http://github.com/poomcj/NPFimg; python npfimg_main.py --input gcms_map.csv --output marker_features.png --mode image_ml
```

## Evaluation signals

- Output visualization of identified marker features is successfully generated and overlaid on the two-dimensional MS map.
- Signal-acquisition error rate from NPFimg is lower than that obtained with XCMS on the same GC–MS or LC–MS data.
- Marker-identification error rate from NPFimg is lower than that obtained with XCMS on the same data.
- Identified multivariate chemo-/biomarker features are consistent with the paper's reported aroma odor and human breath case-study results at parts-per-billion levels.
- No conventional peak-picking artifacts (false peak detections) are present in the discriminated analyte set — NPFimg's core design rationale is to avoid these.

## Limitations

- No changelog is available for the NPFimg repository, making it harder to track feature/version evolution between releases.
- The paper states that while case studies are shown for aroma odor and human breath GC–MS, 'this study shows the limited applications' — broader applicability beyond diverse metabolomics/chemometrics using GC–MS and LC–MS is 'potentially' applicable rather than fully validated.
- Performance at parts-per-billion analyte levels is demonstrated only for the aroma odor and human breath GC–MS case studies; other matrices/concentration regimes are not benchmarked in the article.
- Reliability claim rests on comparison with XCMS; no additional benchmark tools are cited in the provided context.

## Evidence
