---
name: microbiome-metabolite-prediction-modeling
description: Use when paired samples have matched microbiome (taxonomic or functional
  abundance) and metabolite count tables and the goal is to predict metabolite abundances,
  identify well-predicted metabolites, or infer microbe-metabolite interaction networks
  without relying on metabolite annotations.
when_to_use_negative:
- When the goal is a per-metabolite regularized linear model rather than a neural
  network — MelonnPan's Elastic Net trains each metabolite separately and would be
  the alternative choice.
- When input requires UniRef90 gene families from HUMAnN2 for a pre-trained human-gut
  predictor — MelonnPan-Predict is designed for that workflow, not MiMeNet.
- When metabolite features are already transformed and low-abundance filtering would
  destroy signal — pass 'None' normalization instead of re-transforming.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3474
tools:
- name: MiMeNet
  role: Primary neural network framework for predicting metabolomic profiles from
    microbiome data and learning microbe-metabolite relationships
  repo: github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Linear-regression baseline (Elastic Net, per-metabolite) against which MiMeNet's
    multivariate neural network predictions are compared
  repo: biobakery/melonnpan
- name: NED
  role: Neural encoder-decoder baseline used in performance comparisons and IBD (External)
    validation scatterplots
  repo: vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:a6258ebbae9b075adfbcba0230ae5cf67a8da184689e1b31371627001c12342f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MelonnPan
  - MiMeNet
  - NED
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# microbiome-metabolite-prediction-modeling

## Summary

Train a multi-layer perceptron neural network (MiMeNet) that predicts the entire community metabolome profile from metagenomic taxonomic or functional features, and extract well-predicted metabolites plus microbe-metabolite interaction modules via biclustering.

## When to use

Use when paired samples have matched microbiome (taxonomic or functional abundance) and metabolite count tables and the goal is to predict metabolite abundances, identify well-predicted metabolites, or infer microbe-metabolite interaction networks without relying on metabolite annotations.

## When NOT to use

- When the goal is a per-metabolite regularized linear model rather than a neural network — MelonnPan's Elastic Net trains each metabolite separately and would be the alternative choice.
- When input requires UniRef90 gene families from HUMAnN2 for a pre-trained human-gut predictor — MelonnPan-Predict is designed for that workflow, not MiMeNet.
- When metabolite features are already transformed and low-abundance filtering would destroy signal — pass 'None' normalization instead of re-transforming.

## Inputs

- Comma-delimited CSV matrix of samples by microbial features (e.g. data/IBD/microbiome_PRISM.csv)
- Comma-delimited CSV matrix of samples by metabolomic features (e.g. data/IBD/metabolome_PRISM.csv)
- Optional external validation microbiome and metabolome matrices (-external_micro, -external_metab)
- Optional metabolite annotation file (data/IBD/metabolome_annotation.csv) restricting to annotated metabolites
- JSON file of network hyperparameters (layers, layer size, L2 penalty, dropout rate)
- Optional sample labels file to associate clusters with sample conditions

## Outputs

- Per-metabolite Spearman correlation coefficient (SCC) values from cross-validated evaluation
- Well-predicted metabolite list (SCC above the 95th percentile threshold)
- Trained MiMeNet model weights and network hyperparameter report (e.g. results/IBD/network_parameters.txt)
- Biclustered microbe and metabolite modules with shared interaction patterns
- Optional consensus clustering plots when -labels is supplied

## How to apply

Run MiMeNet_train.py with the paired microbiome and metabolite comma-delimited sample-by-feature matrices (e.g. data/IBD/microbiome_PRISM.csv and metabolome_PRISM.csv), supplying network hyperparameters (number of layers, layer size, L2 penalty, dropout rate) through a JSON -net_params file. Apply compositional transformation (RA or CLR, or 'None' if already transformed) and filter low abundant microbial and metabolite features, then train with 10 iterations of 10-fold cross-validation (-num_cv 10 -num_run_cv 10) to obtain per-metabolite Spearman correlation coefficients (SCC). Flag a metabolite as well-predicted when its SCC exceeds the 95th percentile of a shuffled-sample empirical background (>=10 background iterations). For module discovery, extract the microbe-metabolite interaction score matrix and bicluster it into microbe and metabolite modules; optionally supply -labels to associate clusters with sample conditions.

## Related tools

- **MiMeNet** (Primary neural network framework for predicting metabolomic profiles from microbiome data and learning microbe-metabolite relationships) — github.com/YDaiLab/MiMeNet
- **MelonnPan** (Linear-regression baseline (Elastic Net, per-metabolite) against which MiMeNet's multivariate neural network predictions are compared) — biobakery/melonnpan
- **NED** (Neural encoder-decoder baseline used in performance comparisons and IBD (External) validation scatterplots) — vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -micro_norm None -metab_norm CLR -net_params net_params.json -output results/IBD -num_cv 10 -num_run_cv 10 -num_background 10
```

## Evaluation signals

- Mean SCC increases over the linear baseline on each dataset: 0.108 to 0.309 (IBD PRISM), 0.276 to 0.457 (cystic fibrosis), -0.272 to 0.264 (soil)
- Well-predicted metabolite count increases: 198 to 366, 104 to 143, 4 to 29 versus linear models
- Ten iterations of 10-fold cross-validation with num_background >= 10 shuffled-sample iterations produce a stable 95th-percentile SCC threshold
- Training on the full 8848-metabolite output layer raises annotated-subset mean SCC from 0.259 to 0.309 with Wilcoxon signed-rank P < 10^-47 versus annotated-only training
- Robustness check: mean SCC degrades only slightly at reduced k (0.297 to 0.218 at k=2 on IBD PRISM) or 40% sample subsets (0.287 to 0.179)

## Limitations

- Not all metabolites may be associated with microbes, producing lower prediction correlations and an overall lower mean SCC across all metabolites.
- All methods including MiMeNet remain limited in providing biological plausibility and mechanistic insights into microbe-metabolite interactions.
- The soil dataset showed a higher 95th-percentile threshold value, possibly due to longitudinal observations.
- Dependencies are pinned to older versions (Tensorflow 1.14, Python 3.7+); Python 3.7+ is required.

## Evidence

- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [methods] MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [methods] This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns.: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [abstract] MiMeNet more accurately predicts metabolite abundances (mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264): "MiMeNet more accurately predicts metabolite abundances (mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264)"
- [readme] -micro MICRO Comma delimited file representing matrix of samples by microbial features: "-micro MICRO Comma delimited file representing matrix of samples by microbial features"
- [readme] -metab_norm METAB_NORM Metabolome normalization (RA, CLR, or None): "-metab_norm METAB_NORM Metabolome normalization (RA, CLR, or None)"
