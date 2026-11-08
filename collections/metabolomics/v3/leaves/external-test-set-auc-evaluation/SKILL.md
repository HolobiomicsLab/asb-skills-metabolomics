---
name: external-test-set-auc-evaluation
description: Use when a paired microbiome-metabolome dataset has a separate external
  validation set (e.g. IBD (PRISM) training vs. IBD (External) test) and you must
  quantify whether derived module features retain disease-predictive information via
  AUC rather than in-sample cross-validation.
when_to_use_negative:
- Input lacks a genuinely external test set and only cross-validation within a single
  dataset is possible — use the standard cross-validated Spearman correlation evaluation
  (SCC) pipeline instead.
- The task is metabolite abundance prediction accuracy (per-metabolite SCC and well-predicted
  metabolite identification) rather than disease-status classification.
- Modules have not yet been constructed from the interaction-score matrix; the biclustering/consensus
  clustering step must run first before module feature values can be computed.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3697
tools:
- name: MiMeNet
  role: Trains the MLPNN classifiers on original, MiMeNet-module, and WGCNA-module
    feature sets and provides the -external_micro/-external_metab inputs and -labels
    option used to score the external test set
  repo: github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:1b9d9f471b116d6eec4997965cfbd820888a1dc605b15b0eb1f2d7e07b7fec94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# external-test-set-auc-evaluation

## Summary

Evaluate 100-replicate three-layer neural network (MLPNN) classifiers on a held-out external IBD test set by computing area under the ROC curve (AUC) per input feature type, to compare MiMeNet module features against original abundance features and WGCNA module features.

## When to use

Use when a paired microbiome-metabolome dataset has a separate external validation set (e.g. IBD (PRISM) training vs. IBD (External) test) and you must quantify whether derived module features retain disease-predictive information via AUC rather than in-sample cross-validation.

## When NOT to use

- Input lacks a genuinely external test set and only cross-validation within a single dataset is possible — use the standard cross-validated Spearman correlation evaluation (SCC) pipeline instead.
- The task is metabolite abundance prediction accuracy (per-metabolite SCC and well-predicted metabolite identification) rather than disease-status classification.
- Modules have not yet been constructed from the interaction-score matrix; the biclustering/consensus clustering step must run first before module feature values can be computed.

## Inputs

- IBD (PRISM) microbiome count matrix (samples x microbial features, comma-delimited)
- IBD (PRISM) metabolome count matrix (samples x metabolomic features, comma-delimited)
- IBD (External) test-set microbiome and metabolome matrices (comma-delimited)
- diagnosis labels file data/IBD/diagnosis_PRISM.csv mapping sample to IBD class
- MiMeNet microbe and metabolite module assignments (biclusters from interaction-score matrix)
- WGCNA module assignments
- MiMeNet network hyperparameters JSON (via -net_params)

## Outputs

- auc_comparison.csv tabulating AUC distributions per feature type (original, MiMeNet modules, WGCNA modules)
- Per-replicate AUC values for each of the 100 MLPNN training runs

## How to apply

Load the IBD (PRISM) microbiome and metabolome count matrices plus the IBD (External) test-set matrices with pandas and apply the compositional normalization (relative abundance or centered log-ratio) and low-abundance feature filtering used in the MiMeNet run. Compute module feature values as the mean normalized abundance per module for both the MiMeNet biclustered microbe/metabolite modules (from the upstream biclustering of the interaction-score matrix) and the WGCNA modules. Load diagnosis labels from data/IBD/diagnosis_PRISM.csv to associate each sample with its class, then train 100 replicate three-layer MLPNN classifiers using the MiMeNet network hyperparameters (from the -net_params JSON) for each input feature type: original features, MiMeNet module features, and WGCNA module features. Apply each trained classifier to the IBD (External) test set, compute the AUC of the ROC curve per replicate and feature type, and save the AUC distributions to auc_comparison.csv in the output directory. The rationale is that MiMeNet learns metabolites simultaneously so information transfers across metabolites with similar patterns; the external AUC check verifies that this learned representation does not lose disease-predictive signal relative to raw abundance features.

## Related tools

- **MiMeNet** (Trains the MLPNN classifiers on original, MiMeNet-module, and WGCNA-module feature sets and provides the -external_micro/-external_metab inputs and -labels option used to score the external test set) — github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbe_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -external_micro data/IBD/microbe_External.csv -external_metab data/IBD/metabolome_External.csv -labels data/IBD/diagnosis_PRISM.csv -net_params net_params.json -output results_auc -num_run 100
```

## Evaluation signals

- The output auc_comparison.csv contains three groups of AUC values corresponding to original features, MiMeNet module features, and WGCNA module features, each based on 100 replicate runs.
- Reproduced comparison shows MiMeNet module feature AUC is comparable to original-feature AUC on the IBD (External) test set.
- WGCNA module feature models yield a much lower mean AUC than both original and MiMeNet module features (as in Fig 5C).
- The same compositional normalization (RA or CLR) and low-abundance filtering applied to training data is identically applied to the external test matrices.
- External test samples are scored strictly with models trained on PRISM data — no leakage of external labels into training.

## Limitations

- Some metabolites may not be associated with microbes, which lowers overall mean correlation and complicates feature-based interpretation.
- MiMeNet (and compared methods) are limited in providing biological plausibility and mechanistic insight; AUC comparison shows predictive retention, not causality.
- Higher threshold values may be observed for longitudinal datasets (e.g. soil data), so the significant-correlation threshold may need adjustment per dataset.

## Evidence

- [results] Neural network classifiers evaluated on the IBD (External) dataset showed that MiMeNet's modules had predictive power similar to the original features, while models trained on WGCNA module feature values had a much lower mean AUC (Fig 5C).: [evidence span withheld — anchor status: not_found]
- [results] Workflow: load PRISM and external matrices, normalize, compute module means, train 100 replicate MLPNNs, evaluate AUC.: [evidence span withheld — anchor status: not_found]
- [methods] MiMeNet preprocessing applies compositional transformation and low-abundance filtering.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [readme] README documents external_micro and external_metab CLI options for external validation.: "-external_micro EXTERNAL_MICRO    Comma delimited file representing matrix of samples by microbial features"
- [readme] README documents the labels option to associate clusters with sample classes.: "-labels LABELS                    Comma delimited file for sample labels to associate clusters with"
