---
name: metabolome-module-feature-aggregation
description: Use when you have MiMeNet biclustered microbe/metabolite modules and
  WGCNA modules for paired microbiome-metabolome data (e.g. IBD PRISM) and want to
  determine if module-level features match original features for predicting disease
  labels.
when_to_use_negative:
- Input is a single feature table without paired microbiome-metabolome data — this
  skill requires both modalities to define biclustered microbe and metabolite modules
- You only want to predict the metabolomic profile from microbial taxa (use the MiMeNet
  prediction workflow) rather than evaluate phenotype-relevant information retained
  by module features
- No supervised sample labels (e.g. IBD diagnosis) exist — classifier training and
  AUC evaluation cannot be performed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3420
tools:
- name: MiMeNet
  role: Trains the MLPNN classifiers on original, MiMeNet-module, and WGCNA-module
    feature values; produces the interaction-score matrix biclustered into microbe/metabolite
    modules and the module feature aggregation pipeline.
  repo: github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:0535610540e14d3a93081761c7369f185d0f4144ed281bcede8841dde6ef2727
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolome-module-feature-aggregation

## Summary

Aggregates microbe and metabolite bicluster modules into per-module feature values by averaging normalized abundances, then compares their predictive power (via MLPNN classifiers on IBD data) against original features and WGCNA module features to test whether MiMeNet modules retain phenotype-relevant information.

## When to use

Use when you have MiMeNet biclustered microbe/metabolite modules and WGCNA modules for paired microbiome-metabolome data (e.g. IBD PRISM) and want to determine if module-level features match original features for predicting disease labels.

## When NOT to use

- Input is a single feature table without paired microbiome-metabolome data — this skill requires both modalities to define biclustered microbe and metabolite modules
- You only want to predict the metabolomic profile from microbial taxa (use the MiMeNet prediction workflow) rather than evaluate phenotype-relevant information retained by module features
- No supervised sample labels (e.g. IBD diagnosis) exist — classifier training and AUC evaluation cannot be performed

## Inputs

- IBD PRISM microbiome count matrix (samples x microbial features, comma-delimited)
- IBD PRISM metabolome count matrix (samples x metabolomic features, comma-delimited)
- IBD External test-set microbiome and metabolome matrices
- diagnosis_PRISM.csv sample labels associating each sample with its IBD class
- MiMeNet biclustered microbe and metabolite module assignments from the interaction-score matrix
- WGCNA module assignments for the same features

## Outputs

- auc_comparison.csv tabulating AUC distributions for original features, MiMeNet module features, and WGCNA module features
- Mean AUC per feature type (e.g. MiMeNet module features vs WGCNA module features vs original features)

## How to apply

Load the paired microbiome and metabolome count matrices (e.g. IBD PRISM as pandas DataFrames) and the IBD External test-set matrices, then apply the compositional normalization used in the MiMeNet run (relative abundance or CLR) to both modalities. For each microbe and metabolite module from MiMeNet's biclustering of the interaction-score matrix, compute the module feature value as the mean normalized abundance of its member features; repeat the same aggregation for WGCNA modules. Train 100 replicate three-layer MLPNN classifiers (MiMeNet's neural network hyperparameters) on three input types: original feature matrix, MiMeNet module features, and WGCNA module features, using IBD labels from the diagnosis file. Evaluate each classifier on the IBD External test set and record the area under the ROC curve (AUC) per input type, tabulating AUC distributions as auc_comparison.csv in the MiMeNet results directory. The rationale is that MiMeNet learns multiple metabolites simultaneously so underlying information transfers across metabolites, and module features should capture the same predictive signal as raw abundances while WGCNA modules may not.

## Related tools

- **MiMeNet** (Trains the MLPNN classifiers on original, MiMeNet-module, and WGCNA-module feature values; produces the interaction-score matrix biclustered into microbe/metabolite modules and the module feature aggregation pipeline.) — github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/PRISM_micro.csv -metab data/IBD/PRISM_metab.csv -external_micro data/IBD/External_micro.csv -external_metab data/IBD/External_metab.csv -labels data/IBD/diagnosis_PRISM.csv -output results/ -micro_norm RA -metab_norm RA -num_run 100
```

## Evaluation signals

- MiMeNet module feature AUC should be comparable to original feature AUC on the IBD External test set
- WGCNA module feature AUC should be substantially lower than MiMeNet module feature AUC (mean AUC comparison)
- auc_comparison.csv must contain AUC values for all three input types: original features, MiMeNet module features, and WGCNA module features
- Each of the 100 replicate classifiers should be trained and evaluated independently; variability of AUC across replicates is expected
- Module feature values must equal the mean of normalized abundances for members of each module (verify against RA or CLR transformed input)

## Limitations

- All methods including MiMeNet are limited in providing biological plausibility and mechanistic insights
- The comparison relies on an external IBD test set; if no external validation set exists, AUC evaluation is restricted to cross-validation within the discovery cohort
- MiMeNet module feature aggregation assumes the module assignments (from biclustering) are stable across data transformations; consensus clustering was used to determine cluster number for the IBD PRISM dataset

## Evidence

- [results] MiMeNet module features retain predictive information comparable to original abundance features, while WGCNA module features had a much lower mean AUC: [evidence span withheld — anchor status: not_found]
- [methods] Module feature values computed as mean normalized abundance per module: [evidence span withheld — anchor status: not_found]
- [methods] Training 100 replicate three-layer neural network classifiers using MiMeNet hyperparameters: [evidence span withheld — anchor status: not_found]
- [methods] Compositional transformation to RA or CLR and filtering low abundant features: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [methods] Biclustering interaction score matrix into microbe and metabolite modules: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [intro] Learning multiple metabolites simultaneously transfers information across metabolites: "by learning multiple metabolites simultaneously, the underlying information can be transferred to augment the learning of metabolites with similar patterns [30], thus leading to more robust"
- [readme] MiMeNet package and datasets freely available at GitHub: "The MiMeNet package and the datasets used for evaluation are freely available at https://github.com/YDaiLab/MiMeNet"
