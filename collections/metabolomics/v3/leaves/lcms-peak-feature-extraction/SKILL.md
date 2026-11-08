---
name: lcms-peak-feature-extraction
description: Use when raw nontargeted LC-MS metabolomics data (e.g., human plasma
  HILIC runs) must be converted into peak lists/peak groups before ML-based true vs
  false peak filtering.
when_to_use_negative:
- The input is already a filtered feature table or peak list — this skill targets
  raw LC-MS data needing initial peak extraction
- The goal is conventional quality filtering only (centWave-style quality cutoffs)
  rather than building candidates for machine learning peak classification
- Data are not nontargeted LC-MS metabolomics peak-picking outputs (e.g., proteomics
  or genomics data outside MZmine 2 workflows)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Peak extraction of raw LC-MS data into non-filtered peak lists using More/Less
    Restrictive parameters
- name: centWave
  role: Conventional peak detection algorithm used within MZmine-based processing
    for quality filtering/detection of peaks
- name: R
  role: Custom R scripts implementing downstream peak group parameter models on the
    extracted peak lists
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:bb3638edff0d3e0016b23c7d98f1c97a0940090bdd4ca66ed2023e069ecbc521
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - R
  - centWave
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# lcms-peak-feature-extraction

## Summary

Extracts unfiltered LC-MS spectral peak lists from raw metabolomics data using MZmine 2 (v2.14) centWave/local-minimum-search-based peak picking under both 'More Restrictive' and 'Less Restrictive' parameter sets, producing candidate peak groups for downstream true/false classification.

## When to use

Use when raw nontargeted LC-MS metabolomics data (e.g., human plasma HILIC runs) must be converted into peak lists/peak groups before ML-based true vs false peak filtering.

## When NOT to use

- The input is already a filtered feature table or peak list — this skill targets raw LC-MS data needing initial peak extraction
- The goal is conventional quality filtering only (centWave-style quality cutoffs) rather than building candidates for machine learning peak classification
- Data are not nontargeted LC-MS metabolomics peak-picking outputs (e.g., proteomics or genomics data outside MZmine 2 workflows)

## Inputs

- Raw nontargeted HILIC LC-MS metabolomics datasets (e.g., human plasma from Cohort 1 [N=78] and Cohort 2 [N=526], downloadable from https://doi.org/doi:10.25345/C5FD2F)
- MZmine 2 (v2.14) 'More Restrictive' and 'Less Restrictive' peak extraction parameter sets

## Outputs

- Non-filtered peak lists (candidate peak groups) per parameter setting
- Peak group parameter tables (e.g., 59 peak group parameters incl. medRelFWHM) for ML filtering
- Stacked peak plots / raster images of peak shapes across the 63 most-intense samples

## How to apply

Process raw LC-MS files in MZmine 2 (v2.14) using conventional peak extraction algorithms such as centWave or local minimum search, generating non-filtered peak lists with both 'More Restrictive' and 'Less Restrictive' parameter sets so that downstream filters can be compared (more restrictive settings yielded ~11,026 features at ~50% true/~50% false; less restrictive settings captured more peaks at ~75% false positive proportion). Keep the full unfiltered peak list rather than applying quality cutoffs, since the point is to supply candidate peak groups to the image-based deep neural network (script EDML_deep_learning2.py, threshold 0.4725) or the 15-parameter multiple logistic regression model. Optionally generate stacked peak plots per feature from the 63 samples where each candidate peak is most intense, converting peak shape information into raster images (pixel = intensity, row = sample, column = time) for the neural network. Write extracted peak lists and peak group parameter tables for classification and validation.

## Related tools

- **MZmine 2** (Peak extraction of raw LC-MS data into non-filtered peak lists using More/Less Restrictive parameters)
- **centWave** (Conventional peak detection algorithm used within MZmine-based processing for quality filtering/detection of peaks)
- **R** (Custom R scripts implementing downstream peak group parameter models on the extracted peak lists)

## Examples

```
python EDML_deep_learning2.py  # after downloading datasets from https://doi.org/doi:10.25345/C5FD2F and extracting raw data with MZmine 2 (v2.14) More/Less Restrictive parameters
```

## Evaluation signals

- Under More Restrictive settings, extraction should capture roughly 11,026 features with an approximately 50/50 true/false positive split before ML filtering
- Under Less Restrictive settings, total peaks increase but the false positive proportion rises to ~75%, providing the candidate pool for ~90% false-peak removal by ML filters
- Every extracted peak group has 59 peak group parameters computed (with medRelFWHM among the most predictive), enabling downstream logistic regression filtering
- Stacked peak plots can be generated from the 63 samples where each candidate peak is most intense, and converted to raster images for the neural network
- Downstream DNN scoring at threshold 0.4725 should yield ~0.945 true positive rate and ~0.04 false positive rate, indicating extraction preserved true peak shape information

## Limitations

- Extraction alone cannot distinguish true from false peaks: conventional processing at More Restrictive settings still yields ~50% false positives, and Less Restrictive settings yield ~75% false positives, so ML filtering is required
- Parameter settings trade off sensitivity vs false positive burden — no single restrictive setting removes false peaks without losing true positives
- Peak shape rasterization depends on per-sample intensity information from the 63 most-intense samples; peaks absent or noisy in those samples may produce poor images

## Evidence

- [results] Non-filtered peak lists were then generated using MZmine 2 where the raw data was extracted using both "More Restrictive" and "Less Restrictive" peak extraction parameters: [evidence span withheld — anchor status: not_found]
