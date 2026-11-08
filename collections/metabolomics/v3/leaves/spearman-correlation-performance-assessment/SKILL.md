---
name: spearman-correlation-performance-assessment
description: Use when a paired microbiome-metabolome model (e.g. MiMeNet, MelonnPan,
  NED) has been trained and cross-validated and you need per-metabolite SCC values
  plus a shuffled-data empirical background to declare which metabolites are well-predicted.
when_to_use_negative:
- Data is not paired — microbial and metabolite features come from different samples;
  SCC comparison requires paired samples.
- You need p-value-based significance testing on individual microbe-metabolite associations;
  MiMeNet's SCC assessment flags metabolites that are predictable, not statistically
  significant pairwise correlations.
- Only a small number of cross-validated iterations is available (e.g. num_run_cv
  < 10 or num_background < 10); the 95th-percentile background is unstable with too
  few shuffled iterations.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3339
tools:
- name: MiMeNet
  role: Computes per-metabolite SCC via cross-validated evaluation and generates the
    shuffled background used to declare well-predicted metabolites.
  repo: github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Baseline linear model whose cross-validated SCC values are compared against
    MiMeNet's to show performance gains.
  repo: biobakery/melonnpan
- name: NED
  role: Baseline encoder-decoder neural model whose SCC values on the IBD (External)
    dataset are compared with MiMeNet's.
  repo: vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:ff20143f6c2c17e8112469cb101b883a7091ac4dc2e47f2f4b3834b215a7bbc6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MelonnPan
  - MiMeNet
  - NED
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spearman-correlation-performance-assessment

## Summary

Assess per-metabolite prediction performance by computing Spearman correlation coefficients (SCC) between predicted and observed metabolite abundances under repeated k-fold cross-validation, and flag well-predicted metabolites as those whose true SCC exceeds the 95th percentile of an empirically shuffled background distribution.

## When to use

Use when a paired microbiome-metabolome model (e.g. MiMeNet, MelonnPan, NED) has been trained and cross-validated and you need per-metabolite SCC values plus a shuffled-data empirical background to declare which metabolites are well-predicted.

## When NOT to use

- Data is not paired — microbial and metabolite features come from different samples; SCC comparison requires paired samples.
- You need p-value-based significance testing on individual microbe-metabolite associations; MiMeNet's SCC assessment flags metabolites that are predictable, not statistically significant pairwise correlations.
- Only a small number of cross-validated iterations is available (e.g. num_run_cv < 10 or num_background < 10); the 95th-percentile background is unstable with too few shuffled iterations.

## Inputs

- Cross-validated SCC values per metabolite produced by MiMeNet_train.py (num_run_cv >= 10, num_cv >= 5)
- Background SCC distribution generated from randomly shuffled sample labels for the same number of cross-validated iterations
- CSV files of paired microbial counts (samples x microbial features) and metabolite counts (samples x metabolomic features), optionally with external test-set CSVs

## Outputs

- Per-metabolite Spearman correlation coefficients (SCC) from cross-validated evaluation
- List of well-predicted metabolites: those with SCC above the 95th percentile of the shuffled background (or above a user-specified -threshold cutoff)
- Mean SCC summaries for method/dataset comparison (e.g. 0.309 vs 0.108 on IBD)

## How to apply

Run the trained MiMeNet model (multi-layer perceptron with hyper-parameters — number of layers, layer size, L2 penalty, dropout rate — taken from a JSON net_params file) through MiMeNet_train.py with num_run_cv=10 iterations of k-fold cross-validation (num_cv at least 5; the paper uses 10). For each metabolite, compute the Spearman correlation coefficient between predicted and observed abundance across held-out folds, yielding one SCC per metabolite per iteration. In parallel, generate a background of SCC values by running the same cross-validated pipeline with samples randomly shuffled for each iteration (num_background at least 10). Flag a metabolite as well-predicted only if its observed SCC exceeds the 95th percentile of the shuffled background distribution, or apply a pre-defined correlation cutoff via the -threshold flag. Compare mean SCCs across methods and datasets (paper baselines: MiMeNet 0.309 vs MelonnPan 0.108 on IBD, 0.457 vs 0.276 on cystic fibrosis, 0.264 vs -0.272 on soil).

## Related tools

- **MiMeNet** (Computes per-metabolite SCC via cross-validated evaluation and generates the shuffled background used to declare well-predicted metabolites.) — github.com/YDaiLab/MiMeNet
- **MelonnPan** (Baseline linear model whose cross-validated SCC values are compared against MiMeNet's to show performance gains.) — biobakery/melonnpan
- **NED** (Baseline encoder-decoder neural model whose SCC values on the IBD (External) dataset are compared with MiMeNet's.) — vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -micro_norm None -metab_norm CLR -num_run_cv 10 -num_cv 10 -output IBD
```

## Evaluation signals

- Mean SCC from cross-validated evaluation should match paper-reported values (e.g. 0.309, 0.457, 0.264 for MiMeNet; 0.108, 0.276, -0.272 for MelonnPan on the three datasets).
- The number of well-predicted metabolites should increase relative to linear baselines (paper: 198→366, 104→143, 4→29).
- Background distribution must be produced with samples randomly shuffled for every cross-validated iteration, not just once.
- Well-predicted metabolite set must equal those with SCC above the 95th percentile of the shuffled background (or above the user-defined -threshold).
- SCC values should be present for every metabolite feature in the metabolome input table, not only annotated ones.

## Limitations

- Not all metabolites may be associated with microbes, so some will have low SCC and the overall mean correlation across all metabolites will be lower.
- The soil dataset showed a higher correlation threshold, possibly due to longitudinal observations (unresolved).
- SCC assessment identifies predictable metabolites but does not by itself provide biological plausibility or mechanistic insight into microbe-metabolite interactions.

## Evidence

- [methods] Cross-validated SCC evaluation: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite representing how well they could be predicted."
- [methods] 95th percentile background cutoff: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [methods] Shuffled background generation: "However, to generate the background distribution of SCCs, the samples are randomly shuffled for each cross-validated iteration."
- [abstract] Paper-reported SCC results: "MiMeNet more accurately predicts metabolite abundances (mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264)"
- [readme] Cross-validation iteration parameters: "num_background| Integer for number of iterations of 10-fold cross-validation to run on shuffled data in order to generate empirical background (Recommend at least 10)"
