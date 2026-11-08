---
name: microbiome-module-feature-engineering
description: Use when you have paired microbiome-metabolite data and want to replace
  thousands of original abundance features with a smaller set of module feature values
  (mean normalized abundance per biclustered or WGCNA-derived module) for a classification
  task such as IBD status prediction.
when_to_use_negative:
- You only need the predicted metabolomic profile itself — use MiMeNet's prediction
  workflow rather than extracting module features for classification.
- Your input feature table is already a low-dimensional set of module scores, so compressing
  further adds no value.
- The goal is mechanistic discovery of microbe-metabolite interaction modules rather
  than building a classification feature set.
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0203
tools:
- name: MiMeNet
  role: Provides the compositional normalization, biclustered microbe/metabolite modules,
    and the module feature engineering pipeline used as the comparison basis
  repo: https://github.com/YDaiLab/MiMeNet
- name: WGCNA
  role: Baseline module assignments used to derive module feature values for comparison
    against MiMeNet modules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:9d849c6c971667edd03f0925817632578ca962b372d0de89904fdbe33c15b068
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  - WGCNA
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# microbiome-module-feature-engineering

## Summary

Engineering compact module-level features (mean normalized abundance per microbe/metabolite module) for downstream classification, as used in MiMeNet's comparison of its biclustered modules against WGCNA module features and original abundance features for IBD prediction.

## When to use

Use when you have paired microbiome-metabolite data and want to replace thousands of original abundance features with a smaller set of module feature values (mean normalized abundance per biclustered or WGCNA-derived module) for a classification task such as IBD status prediction.

## When NOT to use

- You only need the predicted metabolomic profile itself — use MiMeNet's prediction workflow rather than extracting module features for classification.
- Your input feature table is already a low-dimensional set of module scores, so compressing further adds no value.
- The goal is mechanistic discovery of microbe-metabolite interaction modules rather than building a classification feature set.

## Inputs

- Comma-delimited sample x microbe abundance matrix (training set, e.g. IBD PRISM)
- Comma-delimited sample x metabolite abundance matrix (training set)
- External test-set microbe and metabolite matrices (e.g. IBD External)
- Module assignments: MiMeNet biclustered microbe/metabolite modules (from biclustering the interaction-score matrix) and/or WGCNA module assignments
- Sample diagnosis labels file (e.g. data/IBD/diagnosis_PRISM.csv)

## Outputs

- Module feature value matrix (mean normalized abundance per module per sample)
- AUC distributions per feature type (original features vs. MiMeNet module features vs. WGCNA module features), e.g. auc_comparison.csv
- Trained MLPNN classifiers per feature type

## How to apply

Load the paired microbiome and metabolome count matrices (e.g. IBD PRISM training and IBD External test sets from MiMeNet) plus the module assignments produced by MiMeNet's biclustering of the interaction-score matrix, or by WGCNA. Apply the same compositional normalization used in the upstream MiMeNet run (relative abundance or CLR, controlled via the -micro_norm / -metab_norm flags with values RA, CLR, or None), then for each sample compute the mean normalized abundance across the microbes (or metabolites) assigned to each module. These module feature vectors are the inputs for a supervised classifier; in the MiMeNet paper the downstream test is a three-layer MLPNN trained with 100 replicates per feature type, using the sample diagnosis labels file (data/IBD/diagnosis_PRISM.csv) to associate each sample with its IBD class. Evaluate each feature type by predicting on the external test set and computing AUC, then tabulate AUC distributions per feature type (e.g. auc_comparison.csv). The rationale is that MiMeNet modules encode shared interaction patterns, so they should retain predictive information (comparable AUC to original features) whereas WGCNA modules show a much lower mean AUC in the IBD External comparison (Fig 5C).

## Related tools

- **MiMeNet** (Provides the compositional normalization, biclustered microbe/metabolite modules, and the module feature engineering pipeline used as the comparison basis) — https://github.com/YDaiLab/MiMeNet
- **WGCNA** (Baseline module assignments used to derive module feature values for comparison against MiMeNet modules)

## Examples

```
python MiMeNet_train.py -micro data/IBD/PRISM_micro.csv -metab data/IBD/PRISM_metab.csv -external_micro data/IBD/External_micro.csv -external_metab data/IBD/External_metab.csv -labels data/IBD/diagnosis_PRISM.csv -output results/auc_comparison -micro_norm RA -metab_norm RA -num_run 100
```

## Evaluation signals

- Reproduced comparison should show MiMeNet module features achieve AUC comparable to original features, while WGCNA module features yield substantially lower AUC on the IBD External test set (mirroring Fig 5C).
- Module feature values must be computed from the same normalization (RA or CLR) as the upstream MiMeNet run to avoid distribution mismatch with the classifier.
- The mean normalized abundance per module per sample matrix must have the same sample dimensionality as the input abundance matrix.
- AUC distributions per feature type should be computed over 100 MLPNN replicate runs so the spread is representative.
- Sanity-check that metabolite-annotated modules (if present) behave consistently with the full metabolome feature set, not just the annotated subset.

## Limitations

- Modules are learned on a specific dataset; their predictive value on an external dataset can drop (as seen for WGCNA modules), so module features are not guaranteed to be transferable.
- The comparison relies on the same hyperparameters and cross-validation protocol as the MiMeNet run; changing them changes the module feature engineering basis.
- Feature engineering depends on the quality of the biclustering of the interaction-score matrix; poorly separated modules can hurt downstream classifier performance.

## Evidence

- [results] Neural network classifiers evaluated on the IBD (External) dataset showed that MiMeNet's modules had predictive power similar to the original features, while models trained on WGCNA module feature values had a much lower mean AUC (Fig 5C).: [evidence span withheld — anchor status: not_found]
- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [methods] This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns.: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [readme] Comma delimited file representing matrix of samples by microbial features: "-micro MICRO                      Comma delimited file representing matrix of samples by microbial features"
- [readme] Microbiome normalization (RA, CLR, or None): "-micro_norm MICRO_NORM            Microbiome normalization (RA, CLR, or None)"
