---
name: elastic-net-metabolite-abundance-prediction
description: Use when paired microbiome-metabolome data (samples x taxonomic/functional
  features, samples x metabolite abundances) is available and per-metabolite Elastic
  Net linear regression predictions with Spearman correlation evaluation are needed
  as a benchmark or standalone predictor.
when_to_use_negative:
- The goal is to learn a multi-metabolite predictive model that transfers information
  across metabolites with similar abundance patterns — MiMeNet's multi-task neural
  network is the appropriate comparator, since per-metabolite linear models miss cross-metabolite
  signal sharing.
- The metabolite is predicted to have low microbial association — the article notes
  some metabolites have lower prediction correlations because not all metabolites
  may be associated with microbes, yielding unreliable SCC for those features.
- Input microbial features are not in UniRef90 gene family format profiled by HUMAnN2
  — the default MelonnPan-Predict pre-trained model requires this specific functional
  profile; otherwise a custom MelonnPan-Train workflow must first be run.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MelonnPan
  role: Per-metabolite Elastic Net linear regression tool for predicting metabolite
    relative abundances from microbial taxonomic or functional features; the linear-model
    baseline being benchmarked in this skill.
  repo: biobakery/melonnpan
- name: MiMeNet
  role: Source of the shared cross-validation protocol, preprocessing (RA/CLR transformation),
    and well-predicted-metabolite thresholding used to benchmark MelonnPan fairly;
    its repository also provides the paired IBD, cystic fibrosis, and soil datasets.
  repo: github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:6e1765a0e58e4183ebeaa1df712c12f725387cc3b4a27054fd1be9be90032c30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/melonnpan
  tools:
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# elastic-net-metabolite-abundance-prediction

## Summary

Baseline benchmarking skill for MelonnPan's per-metabolite Elastic Net linear regression, which predicts metabolite relative abundances from microbial taxonomic or functional features and serves as the linear-model comparator to MiMeNet's multi-task neural network.

## When to use

Use when paired microbiome-metabolome data (samples x taxonomic/functional features, samples x metabolite abundances) is available and per-metabolite Elastic Net linear regression predictions with Spearman correlation evaluation are needed as a benchmark or standalone predictor.

## When NOT to use

- The goal is to learn a multi-metabolite predictive model that transfers information across metabolites with similar abundance patterns — MiMeNet's multi-task neural network is the appropriate comparator, since per-metabolite linear models miss cross-metabolite signal sharing.
- The metabolite is predicted to have low microbial association — the article notes some metabolites have lower prediction correlations because not all metabolites may be associated with microbes, yielding unreliable SCC for those features.
- Input microbial features are not in UniRef90 gene family format profiled by HUMAnN2 — the default MelonnPan-Predict pre-trained model requires this specific functional profile; otherwise a custom MelonnPan-Train workflow must first be run.

## Inputs

- Comma-delimited matrix of samples by microbial sequence features' relative abundances (samples in rows)
- Comma-delimited matrix of metabolite relative abundances (samples in rows)

## Outputs

- MelonnPan_Training_Summary.txt: per-compound prediction accuracy (correlation coefficient), p-value, and q-value for significant metabolites
- MelonnPan_Trained_Metabolites.txt: predicted relative abundances of statistically significant metabolites
- MelonnPan_Trained_Weights.txt: coefficient estimates (Elastic Net weights) per compound
- Per-dataset mean Spearman correlation coefficients and well-predicted metabolite counts (e.g. melonnpan_scc_comparison.csv)

## How to apply

Load comma-delimited matrices of samples by microbial features and samples by metabolomic features, both normalized to relative abundance (RA) — MelonnPan requires RA-transformed input as specified in the MiMeNet comparison protocol. Fit MelonnPan Elastic Net linear regression independently for each metabolite, using the identical 10 iterations of 10-fold cross-validation partitions shared with MiMeNet, tuning hyperparameters on the first partition and applying the shared settings across all iterations. For each metabolite compute the Spearman correlation coefficient (SCC) between predicted and observed abundances on held-out folds; count well-predicted metabolites as those whose SCC exceeds the 95th percentile of a shuffled-background distribution generated by randomly shuffling samples per cross-validated iteration. Aggregate mean SCCs and well-predicted metabolite counts per dataset and compare against MiMeNet's multi-task neural network results, which raised mean SCC from 0.108 to 0.309 (IBD PRISM), 0.276 to 0.457 (cystic fibrosis), and -0.272 to 0.264 (soil).

## Related tools

- **MelonnPan** (Per-metabolite Elastic Net linear regression tool for predicting metabolite relative abundances from microbial taxonomic or functional features; the linear-model baseline being benchmarked in this skill.) — biobakery/melonnpan
- **MiMeNet** (Source of the shared cross-validation protocol, preprocessing (RA/CLR transformation), and well-predicted-metabolite thresholding used to benchmark MelonnPan fairly; its repository also provides the paired IBD, cystic fibrosis, and soil datasets.) — github.com/YDaiLab/MiMeNet

## Examples

```
Rscript -e "library(MelonnPan); melonnpan.train(metab = 'metab.csv', seq = 'micro.csv')"
```

## Evaluation signals

- Mean SCC per dataset must match the MelonnPan reference values reported by MiMeNet's benchmark: 0.108 (IBD PRISM annotated metabolites), 0.276 (cystic fibrosis), -0.272 (soil), and 0.168 (IBD External) within reproducibility tolerance of the 10x 10-fold CV scheme.
- Well-predicted metabolite counts must match the reference: 198 (IBD PRISM), 104 (cystic fibrosis), 4 (soil), and 186 (IBD External), where 'well-predicted' means SCC above the 95th percentile of a shuffled-background distribution.
- Output table MelonnPan_Training_Summary.txt must include per-compound prediction accuracy (correlation coefficient) plus p-value and q-value for each significant metabolite.
- MelonnPan_Trained_Weights.txt must contain non-zero Elastic Net coefficient estimates for at least the significant compounds, confirming the model was fitted per metabolite rather than globally.
- Cross-validation partitions must be identical to those used by MiMeNet (10 iterations of 10-fold CV, hyperparameters tuned from the first partition and shared across iterations).

## Limitations

- Elastic Net linear regression models each metabolite independently and cannot transfer information across metabolites with similar patterns, limiting performance on datasets where cross-metabolite structure is informative.
- Relying on annotated references (as in linear modeling pipelines) can lead to poor-performing models when annotations are missing or incorrect, and hampers discovery of novel mechanisms.
- Not all metabolites may be associated with microbes, so some metabolites will have lower prediction correlations, resulting in an overall lower mean SCC across all metabolites.
- A higher threshold value was observed for the soil data, possibly due to longitudinal observations, complicating well-predicted-metabolite identification for longitudinal studies.
- MelonnPan currently requires input data specified using UniRef90 gene families profiled by HUMAnN2 for its default pre-trained model.

## Evidence

- [intro] MelonnPan uses the Elastic Net linear regression to model the relative abundance of each metabolite using metagenomic taxonomic or functional features: "One method, MelonnPan, uses the Elastic Net linear regression to model the relative abundance of each metabolite using metagenomic taxonomic or functional features"
- [readme] MelonnPan requires RA-transformed relative abundance input: [evidence span withheld — anchor status: not_found]
- [readme] Training workflow requires metabolite and microbial relative abundance tables: [evidence span withheld — anchor status: not_found]
- [readme] Outputs include training summary with per-compound accuracy, trained metabolites, and weights: [evidence span withheld — anchor status: not_found]
- [readme] MelonnPan requires UniRef90 gene families from HUMAnN2: [evidence span withheld — anchor status: external_doc]
- [supplementary] MiMeNet benchmark protocol: 10 iterations of 10-fold cross-validation with shared hyperparameters tuned from first partition: "Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition"
- [methods] Well-predicted metabolites are those with SCC above the 95th percentile: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [methods] Preprocessing includes compositional transformation and filtering of low-abundance features: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [discussion] MelonnPan uses regularized linear regression per metabolite, contrasting with MiMeNet's joint model: "The predictive model in MiMeNet distinguishes it from MelonnPan [26], which uses a regularized linear regression to model each metabolite separately."
- [discussion] Not all metabolites may be associated with microbes, causing lower correlations: "since not all metabolites may be associated with microbes, some metabolites will have lower prediction correlations, which resulted in an overall lower mean correlation across all metabolites."
- [discussion] Higher threshold for soil data possibly due to longitudinal observations: "We also observed a higher threshold value for the soil data (Fig 3A–3C), which may be due to the longitudinal observations."
