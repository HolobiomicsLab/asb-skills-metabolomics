---
name: multivariate-chemomarker-feature-identification
description: Use when raw two-dimensional MS maps (m/z vs retention time) from GC–MS
  or LC–MS must be characterized, analytes discriminated, and multivariate chemo-/biomarker
  features identified and visualized without a peak picking stage.
when_to_use_negative:
- The pipeline output is already a peak-picked feature table (e.g. XCMS output) —
  NPFimg's design deliberately replaces the conventional peak picking process, so
  feeding it pre-picked features defeats its purpose.
- The data are not a two-dimensional chromatography–MS map arranged as m/z versus
  retention time (e.g. standalone direct-injection MS or non-chromatographic spectra),
  which the article does not claim NPFimg handles.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: NPFimg
  role: 'Primary pipeline: image processing plus machine learning over the two-dimensional
    MS map (m/z vs retention time) to discriminate analytes, identify multivariate
    chemo-/biomarker features, and visualize them'
  repo: http://github.com/poomcj/NPFimg
- name: XCMS
  role: External peak-picking-based baseline used for reliability comparison; NPFimg's
    lower error rates of signal acquisition and marker identification are judged against
    XCMS
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:f1659ca2d8dbf84686f0286b50afa63a63371c80521392a017aab576e64fc9b2
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
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

# multivariate-chemomarker-feature-identification

## Summary

Identifies and visualizes multivariate chemo-/biomarker features of analytes in chromatography–MS data (e.g. GC–MS m/z vs retention time maps) using the NPFimg combination of image processing and machine learning, avoiding the false peak detections of conventional peak picking.

## When to use

Use when raw two-dimensional MS maps (m/z vs retention time) from GC–MS or LC–MS must be characterized, analytes discriminated, and multivariate chemo-/biomarker features identified and visualized without a peak picking stage.

## When NOT to use

- The pipeline output is already a peak-picked feature table (e.g. XCMS output) — NPFimg's design deliberately replaces the conventional peak picking process, so feeding it pre-picked features defeats its purpose.
- The data are not a two-dimensional chromatography–MS map arranged as m/z versus retention time (e.g. standalone direct-injection MS or non-chromatographic spectra), which the article does not claim NPFimg handles.

## Inputs

- Two-dimensional chromatography–MS map (m/z vs retention time) from GC–MS or LC–MS acquisitions (e.g. the repository's example two-dimensional GC–MS MS map)
- Sample/condition-grouped MS acquisitions where multivariate chemo-/biomarker features must be discriminated (e.g. aroma odor and human breath GC–MS datasets)

## Outputs

- Set of automatically identified multivariate chemo-/biomarker features of analytes
- Marker-feature visualization image showing the identified features on the two-dimensional MS map

## How to apply

Read the chromatography–MS acquisition as a two-dimensional MS map whose axes are m/z versus retention time and feed it into NPFimg (https://github.com/poomcj/NPFimg, MIT license). First apply the image-processing stage to the MS map image so that signals are comprehensively characterized directly in the image domain, bypassing the conventional peak picking process that suffers from false peak detections. Then apply the machine-learning stage to the processed map to discriminate analytes and automatically identify multivariate chemo-/biomarker features. Generate NPFimg's marker-feature visualization showing the identified features on the two-dimensional MS map as the final output. Judge correctness by (a) the visualization being produced with marker features rendered on the MS map, and (b) error rates of signal acquisition and marker identification being low — the article's reliability benchmark is comparison with XCMS, in which NPFimg shows lower error rates for both. Marker identification has been demonstrated at the parts per billion level on aroma odor and human breath GC–MS case studies, so trace-level features should be expected to be recoverable rather than filtered out.

## Related tools

- **NPFimg** (Primary pipeline: image processing plus machine learning over the two-dimensional MS map (m/z vs retention time) to discriminate analytes, identify multivariate chemo-/biomarker features, and visualize them) — http://github.com/poomcj/NPFimg
- **XCMS** (External peak-picking-based baseline used for reliability comparison; NPFimg's lower error rates of signal acquisition and marker identification are judged against XCMS)

## Examples

```
git clone http://github.com/poomcj/NPFimg  # run NPFimg on the repository's example two-dimensional GC–MS map per the article: pubs.acs.org/doi/full/10.1021/acs.analchem.1c03163
```

## Evaluation signals

- A marker-feature visualization image is produced and the identified marker features are rendered on the two-dimensional MS map
- Error rate of signal acquisition is lower than the corresponding XCMS run on the same GC–MS data
- Error rate of marker identification is lower than the corresponding XCMS run on the same GC–MS data
- Trace-level (parts per billion) analyte features in aroma odor or human breath GC–MS case studies are still identified rather than missed
- No conventional peak picking stage was invoked; signal characterization occurred in the image domain of the MS map

## Limitations

- The README notes documentation is incomplete: 'For the other details, I will update soon' — parameter specifics must be taken from the article at https://pubs.acs.org/doi/full/10.1021/acs.analchem.1c03163
- Demonstrated scope is GC–MS (aroma odor, human breath); applicability to LC–MS and other metabolomics/chemometrics data is stated as potential ('potentially applicable'), not validated in the study
- The authors note 'While this study shows the limited applications', so case-study breadth is narrow despite the reported XCMS comparison

## Evidence
