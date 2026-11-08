---
name: predictive-model-auc-evaluation
description: Use when a tuned supervised classifier (e.g. SKlearn random forest on
  pathway scores) has been trained with nested cross-validation and you must quantify
  held-out predictive performance as AUC, especially to compare against a molecular-level
  concatenated baseline.
when_to_use_negative:
- The analysis goal is unsupervised pathway ranking or conventional ORA/GSEA rather
  than outcome prediction — PathIntegrate's predictive AUC evaluation does not apply
  to MultiGSEA, IMPaLA, or ActivePathways-style hypothesis-generation workflows.
- Input omics are not measured on the same individuals (samples with missing entire
  omics views must be discarded, since PathIntegrate requires all input omics on the
  same individuals).
- The comparison is against methods like DIABLO operating at the molecular level without
  pathway transformation — a different benchmarking protocol than this pathway-level
  AUC evaluation.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: PathIntegrate
  role: Framework that produces the multi-omics pathway-transformed dataset and applies
    the SKlearn-compatible predictive model whose held-out AUC is evaluated
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa
  role: Computes the single-sample pathway scores (SVD/PLAGE, min_entity=2) from Reactome
    multi-omics pathways that form the model input features
  repo: github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Provides the RandomForestClassifier, nested cross-validation tuning, and RFECV
    pathway selection whose test-set AUC is the evaluation metric
- name: PathIntegrate_scripts
  role: Contains the benchmarking and application scripts and COVID-19 data needed
    to replicate the Su et al. 2020 AUC evaluation
  repo: cwieder/PathIntegrate_scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:3cb62cdad121c263e694d4edd2245e0109f08ccbfecfdc5ccfad99b3dfd61c6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9629629629629629
n_claims: 27
---

# predictive-model-auc-evaluation

## Summary

Evaluate a pathway-level (or molecular-level baseline) supervised classification model on a held-out test set using AUC, benchmarking multi-omics pathway-transformed predictions against concatenated molecular data as done in the PathIntegrate Su et al. 2020 COVID-19 severity case study.

## When to use

Use when a tuned supervised classifier (e.g. SKlearn random forest on pathway scores) has been trained with nested cross-validation and you must quantify held-out predictive performance as AUC, especially to compare against a molecular-level concatenated baseline.

## When NOT to use

- The analysis goal is unsupervised pathway ranking or conventional ORA/GSEA rather than outcome prediction — PathIntegrate's predictive AUC evaluation does not apply to MultiGSEA, IMPaLA, or ActivePathways-style hypothesis-generation workflows.
- Input omics are not measured on the same individuals (samples with missing entire omics views must be discarded, since PathIntegrate requires all input omics on the same individuals).
- The comparison is against methods like DIABLO operating at the molecular level without pathway transformation — a different benchmarking protocol than this pathway-level AUC evaluation.

## Inputs

- Multi-omics pathway score matrix (samples x multi-omics Reactome pathways, computed via sspa SVD/PLAGE with min_entity=2, automatically standardised)
- Concatenated molecular-level omics abundance matrix (metabolomics + proteomics, log2-transformed, median-imputed, z-scored) as baseline features
- Binary outcome labels (mild vs moderate/severe COVID-19 status) for the 127 Su et al. 2020 samples
- Tuned SKlearn RandomForestClassifier from the inner CV loop

## Outputs

- Held-out test-set AUC for the pathway-level model (expected ~0.95)
- Held-out test-set AUC for the concatenated molecular-level baseline model (expected ~0.98)
- RFECV-selected pathway list (20 pathways, e.g. Interleukin-5/13 signalling)
- Pathway coverage metadata per selected pathway

## How to apply

Set up a nested cross-validation scheme: tune the classifier hyperparameters (e.g. a SKlearn RandomForestClassifier) on the inner folds, then evaluate the tuned model on the held-out outer test set. Apply the pathway scoring transformation (sspa SVD-based PLAGE on Reactome multi-omics pathways, retaining pathways with at least 2 measured molecules — 732 pathways retained in the COVID-19 case study) to the training and test splits, standardising the pathway score matrix as PathIntegrate performs automatic standardisation. Train the tuned random forest on the training pathway scores, optionally refine the feature set with RFECV (20 pathways selected, e.g. Interleukin-5/13 signalling), and record the test-set AUC. Run the identical protocol on the concatenated molecular-level omics abundance matrix (log2-transformed, median-imputed, z-scored per the sspa pre-processing) to obtain the baseline AUC (0.98 in the paper vs 0.95 for the pathway-level model), so that the loss of prediction performance from pathway-level modelling can be quantified. Judge correctness by confirming the pathway model's AUC is consistent with the published value (~0.95) and that exactly 20 pathways appear in the RFECV selection output.

## Related tools

- **PathIntegrate** (Framework that produces the multi-omics pathway-transformed dataset and applies the SKlearn-compatible predictive model whose held-out AUC is evaluated) — https://github.com/cwieder/PathIntegrate
- **sspa** (Computes the single-sample pathway scores (SVD/PLAGE, min_entity=2) from Reactome multi-omics pathways that form the model input features) — github.com/cwieder/py-ssPA
- **SciKitLearn** (Provides the RandomForestClassifier, nested cross-validation tuning, and RFECV pathway selection whose test-set AUC is the evaluation metric)
- **PathIntegrate_scripts** (Contains the benchmarking and application scripts and COVID-19 data needed to replicate the Su et al. 2020 AUC evaluation) — cwieder/PathIntegrate_scripts

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- Pathway-level model test-set AUC falls in the range consistent with the published 0.95 (vs 0.98 for the molecular-level concatenated baseline).
- RFECV selection output contains exactly 20 pathways.
- Selected pathway list includes biologically expected hits such as Interleukin-5/13 signalling.
- Number of multi-omics pathways retained after min_entity=2 filtering equals 732.
- The molecular-level baseline model uses the same nested CV protocol (tuned random forest, held-out test set) so the AUC comparison is apples-to-apples.

## Limitations

- Lack of appropriate benchmarking data with large sample size (n >= 1000) and validated molecular/pathway-level signals makes rigorous evaluation hard; no such ideal dataset was available.
- Comparing molecular vs. pathway-level models is inherently difficult because it is unknown how many molecules in a pathway are differentially abundant at any one time, and pathway definitions and sizes vary between databases.
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics.
- Samples from individuals without data on all omics must be discarded, as PathIntegrate currently does not support entire rows of missing data.

## Evidence

- [other] PathIntegrate Single-View achieved a test-set classification AUC of 0.95, performing similarly to the concatenated molecular-level omics model which achieved an AUC of 0.98: [evidence span withheld — anchor status: not_found]
- [abstract] single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [other] SVD-based PLAGE scoring with at least 2 measured molecules retaining 732 multi-omics pathways: [evidence span withheld — anchor status: not_found]
- [other] RFECV selected 20 pathways including Interleukin-5/13 signalling: [evidence span withheld — anchor status: not_found]
- [readme] kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2]): [evidence span withheld — anchor status: external_doc]
