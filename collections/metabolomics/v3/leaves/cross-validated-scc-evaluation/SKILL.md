---
name: cross-validated-scc-evaluation
description: Use when a paired microbiome-metabolome model must be evaluated per metabolite
  under repeated cross-validation, or when well-predicted metabolites must be counted
  against an empirical background threshold.
when_to_use_negative:
- When a single held-out external validation set, not repeated cross-validation, is
  the required evaluation design
- When the metabolites of interest were not evaluated on the same folds/samples, making
  paired per-metabolite SCC comparison (and Wilcoxon signed-rank testing) invalid
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2269
tools:
- name: MiMeNet
  role: MLPNN whose cross-validated metabolite predictions supply the per-metabolite
    SCC values and well-predicted metabolite counts via the -num_run_cv / -num_cv
    / -num_background CLI options
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:1fc9eaddf25275940850452d2cf55246c05996b4d647a2eec067dd58da77e323
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cross-validated-scc-evaluation

## Summary

Evaluation protocol from MiMeNet that assesses metabolite predictions via 10 iterations of 10-fold cross-validation, computing per-metabolite Spearman correlation coefficients (SCC) and flagging well-predicted metabolites against an empirical shuffled background.

## When to use

Use when a paired microbiome-metabolome model must be evaluated per metabolite under repeated cross-validation, or when well-predicted metabolites must be counted against an empirical background threshold.

## When NOT to use

- When a single held-out external validation set, not repeated cross-validation, is the required evaluation design
- When the metabolites of interest were not evaluated on the same folds/samples, making paired per-metabolite SCC comparison (and Wilcoxon signed-rank testing) invalid

## Inputs

- Comma-delimited samples-by-microbial-features matrix (e.g., data/IBD/microbiome_PRISM.csv)
- Comma-delimited samples-by-metabolomic-features matrix (e.g., data/IBD/metabolome_PRISM.csv)
- Optional annotation file delimiting a subset of metabolite features (e.g., data/IBD/metabolome_annotation.csv)
- JSON file of network hyperparameters (e.g., results/IBD/network_parameters.txt)
- Background directory or num_background setting for empirical SCC thresholds

## Outputs

- Per-metabolite Spearman correlation coefficients (SCC) from cross-validated evaluation
- Count of well-predicted metabolites whose SCC exceeds the 95th percentile of the shuffled background
- Comparison table of paired SCCs and well-predicted counts (e.g., annotated_vs_full_scc_comparison.csv)

## How to apply

Run num_run_cv=10 iterations of num_cv=10-fold cross-validation on the paired microbial/metabolite matrices, evaluating models so that a Spearman correlation coefficient (SCC) is produced for each metabolite. Count a metabolite as well-predicted only when its SCC exceeds the 95th percentile of the shuffled-sample empirical background, generated with at least num_background>=10 background CV iterations. For paired model comparisons (e.g., full-metabolome vs. annotated-only training), collect per-metabolite SCCs on the shared subset and apply a Wilcoxon signed-rank test on the paired SCC values. Use the same preprocessing (RA/CLR normalization, low-abundance filtering) and hyperparameters across the runs being compared so the SCC distributions are directly comparable.

## Related tools

- **MiMeNet** (MLPNN whose cross-validated metabolite predictions supply the per-metabolite SCC values and well-predicted metabolite counts via the -num_run_cv / -num_cv / -num_background CLI options) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -annotation data/IBD/metabolome_annotation.csv -output results/IBD/ -micro_norm None -metab_norm CLR -num_run_cv 10 -num_cv 10 -num_background 10
```

## Evaluation signals

- Annotated-subset mean SCC must reach ~0.309 for full-metabolome training vs ~0.259 for annotated-only training on IBD (PRISM)
- Well-predicted annotated metabolite count must increase from 333 to 366 when training on the full 8848-metabolite output layer
- Wilcoxon signed-rank test on paired annotated-subset SCCs must yield P < 10^-47
- Well-predicted metabolites must be defined strictly as SCC above the 95th percentile of the empirical background, with num_background >= 10
- All compared runs must use identical CV settings (10 iterations of 10-fold) and shared hyperparameters

## Limitations

- Not all metabolites may be associated with microbes, so some metabolites will have lower prediction correlations, lowering the overall mean SCC
- A higher well-predicted threshold value was observed for the soil dataset, possibly due to longitudinal observations (unresolved)
- Evaluation correlation alone provides limited biological plausibility or mechanistic insight

## Evidence

- [abstract] Ten iterations of 10-fold cross-validation on three paired microbiome-metabolome datasets: "Using ten iterations of 10-fold cross-validation on three paired microbiome-metabolome datasets"
- [methods] Cross-validated SCC per metabolite: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [methods] Well-predicted threshold definition: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [readme] Background and CV parameters in README: "-num_background NUM_BACKGROUND    Number of background CV Iterations"
- [discussion] Annotated-metabolite comparison result: "the MiMeNet prediction on the set of the annotated metabolites benefited from including tasks of predicting the rest of the unannotated metabolites"
- [supplementary] Shared hyperparameters across CV runs: "Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition"
