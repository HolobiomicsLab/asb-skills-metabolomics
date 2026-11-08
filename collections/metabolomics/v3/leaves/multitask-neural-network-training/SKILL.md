---
name: multitask-neural-network-training
description: Use when paired microbiome-metabolome data (e.g. samples x microbial
  features plus samples x 8848 metabolite features) are available and metabolites
  should be predicted jointly rather than one-by-one, especially when many metabolites
  are unannotated.
when_to_use_negative:
- The analysis goal is interpretation of individual metabolite-specific regression
  coefficients rather than joint multivariate prediction (MelonnPan-style per-metabolite
  Elastic Net may be preferable).
- The metabolite matrix contains only a handful of features with no unannotated metabolites,
  so no cross-task information transfer is available to exploit.
- Metabolites are not plausibly microbe-associated — the article notes some metabolites
  will have low prediction correlations because not all metabolites are associated
  with microbes.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiMeNet
  role: MLPNN framework that trains the multitask metabolite-prediction network, performs
    cross-validated SCC evaluation, and identifies well-predicted metabolites
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: 'Baseline contrast: per-metabolite regularized (Elastic Net) linear regression
    used to show the value of multitask joint learning'
  repo: https://github.com/biobakery/melonnpan
- name: BiomeNED
  role: Baseline neural encoder-decoder comparison model for metabolite prediction
  repo: https://github.com/vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:b8335056fba41a86c626e47a054bde9e3f4ea68e8b0c6da1fee91b82eeaba8cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - BiomeNED
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multitask-neural-network-training

## Summary

Train a single multi-layer perceptron (MLPNN) that predicts all metabolomic features simultaneously (the full 8848-metabolite output layer in the IBD/PRISM case) instead of modeling each metabolite independently, so information transfers across metabolites with similar patterns. MiMeNet showed this multivariate learning raises annotated-metabolite SCC from 0.259 to 0.309 and well-predicted counts from 333 to 366 (Wilcoxon P < 10^-47).

## When to use

Use when paired microbiome-metabolome data (e.g. samples x microbial features plus samples x 8848 metabolite features) are available and metabolites should be predicted jointly rather than one-by-one, especially when many metabolites are unannotated.

## When NOT to use

- The analysis goal is interpretation of individual metabolite-specific regression coefficients rather than joint multivariate prediction (MelonnPan-style per-metabolite Elastic Net may be preferable).
- The metabolite matrix contains only a handful of features with no unannotated metabolites, so no cross-task information transfer is available to exploit.
- Metabolites are not plausibly microbe-associated — the article notes some metabolites will have low prediction correlations because not all metabolites are associated with microbes.

## Inputs

- Comma-delimited samples-by-microbial-features matrix (e.g. data/IBD/microbiome_PRISM.csv)
- Comma-delimited samples-by-metabolomic-features matrix (e.g. data/IBD/metabolome_PRISM.csv, 8848 metabolites)
- JSON file of network hyperparameters (e.g. results/IBD/network_parameters.txt)
- Optional annotation file marking the subset of annotated metabolites (e.g. data/IBD/metabolome_annotation.csv, 466 metabolites)
- Normalization choices for microbiome (-micro_norm: RA/CLR/None) and metabolome (-metab_norm: RA/CLR)

## Outputs

- Per-metabolite Spearman correlation coefficients (SCC) across cross-validation folds
- Count of well-predicted metabolites (SCC above 95th percentile of shuffled background)
- Trained MiMeNet model and cross-validated evaluation metrics in the -output directory
- Comparison table such as annotated_vs_full_scc_comparison.csv with paired SCCs and Wilcoxon signed-rank P value

## How to apply

Load the paired matrices (e.g. data/IBD/microbiome_PRISM.csv and data/IBD/metabolome_PRISM.csv) with pandas, apply the chosen normalization (RA, CLR, or 'None' via -micro_norm/-metab_norm; the IBD run used 'None' for microbiome and CLR for metabolome), and filter low-abundant microbial and metabolite features. Train the MLPNN on the full metabolite output layer with the tuned hyperparameters JSON (-net_params results/IBD/network_parameters.txt), running num_run_cv=10 iterations of 10-fold cross-validation (num_cv=10). The rationale is that learning all metabolites simultaneously transfers information across metabolites with similar abundance patterns, augmenting prediction even of annotated metabolites — verified by training a second model restricted to the 466 annotated metabolites (data/IBD/metabolome_annotation.csv) and comparing per-metabolite SCCs with a Wilcoxon signed-rank test. Evaluate each metabolite's Spearman correlation coefficient (SCC) across CV folds and mark metabolites above the 95th percentile of the shuffled-sample background (num_background >= 10) as well-predicted.

## Related tools

- **MiMeNet** (MLPNN framework that trains the multitask metabolite-prediction network, performs cross-validated SCC evaluation, and identifies well-predicted metabolites) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Baseline contrast: per-metabolite regularized (Elastic Net) linear regression used to show the value of multitask joint learning) — https://github.com/biobakery/melonnpan
- **BiomeNED** (Baseline neural encoder-decoder comparison model for metabolite prediction) — https://github.com/vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -annotation data/IBD/metabolome_annotation.csv -net_params results/IBD/network_parameters.txt -output results/IBD -micro_norm None -metab_norm CLR -num_run_cv 10 -num_cv 10
```

## Evaluation signals

- Annotated-subset mean SCC increases from 0.259 (annotated-only model) to 0.309 (full-metabolite model) on IBD (PRISM)
- Wilcoxon signed-rank test on paired annotated-subset SCCs yields P < 10^-47
- Well-predicted annotated metabolite count increases from 333 to 366 (SCC > 95th percentile of shuffled background, num_background >= 10)
- Evaluation uses 10 iterations of 10-fold cross-validation (num_run_cv=10, num_cv=10) with shared hyperparameters tuned from the first partition
- Overall multitask performance matches the paper's reported SCC gains on the three datasets (e.g. 0.108 to 0.309 on IBD)

## Limitations

- Some metabolites have lower prediction correlations because not all metabolites may be associated with microbes, lowering the overall mean correlation.
- MiMeNet (like MelonnPan, mmvec, and NED) is limited in providing biological plausibility and mechanistic insight.
- A higher well-predicted threshold was observed for the soil dataset, possibly due to longitudinal observations (unresolved in the paper).

## Evidence

- [intro] by learning multiple metabolites simultaneously, the underlying information can be transferred to augment the learning of metabolites with similar patterns: "by learning multiple metabolites simultaneously, the underlying information can be transferred to augment the learning of metabolites with similar patterns"
- [discussion] annotated metabolites benefited from including unannotated tasks: "the MiMeNet prediction on the set of the annotated metabolites benefited from including tasks of predicting the rest of the unannotated metabolites"
- [methods] cross-validated SCC evaluation: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [methods] 95th percentile well-predicted threshold: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [readme] README CLI invocation: "MiMeNet_train.py [-h] -micro MICRO -metab METAB [-external_micro EXTERNAL_MICRO] [-external_metab EXTERNAL_METAB]"
