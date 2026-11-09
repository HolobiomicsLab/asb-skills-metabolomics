---
name: cross-validated-neural-network-training
description: Use when paired microbiome and metabolome count tables are available
  and the goal is to predict the community metabolome from microbial features with
  cross-validated SCC evaluation and background-shuffled significance testing.
when_to_use_negative:
- The goal is pairwise microbe-metabolite conditional probabilities rather than whole-profile
  prediction — mmvec is the appropriate alternative.
- Only a subset of predictable metabolites with individual per-metabolite regularized
  linear regression is desired — MelonnPan's Elastic Net approach is the alternative.
- Data are not paired (no matched microbiome and metabolome samples per subject/sample),
  so cross-validated metabolite prediction from taxa is not feasible.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MiMeNet
  role: MLPNN framework trained with MiMeNet_train.py to predict metabolomic profiles
    from microbial features under 10 iterations of 10-fold cross-validation
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:570b1c18900a8beffba99750151bac4b7b0cf17e787ad73f9c52f8a759476ce1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-validated-neural-network-training

## Summary

Train MiMeNet, a multi-layer perceptron neural network (MLPNN), to predict metabolomic profiles from microbial taxonomic features, evaluating performance by per-metabolite Spearman correlation coefficients (SCC) under repeated 10-fold cross-validation on paired microbiome-metabolome datasets.

## When to use

Use when paired microbiome and metabolome count tables are available and the goal is to predict the community metabolome from microbial features with cross-validated SCC evaluation and background-shuffled significance testing.

## When NOT to use

- The goal is pairwise microbe-metabolite conditional probabilities rather than whole-profile prediction — mmvec is the appropriate alternative.
- Only a subset of predictable metabolites with individual per-metabolite regularized linear regression is desired — MelonnPan's Elastic Net approach is the alternative.
- Data are not paired (no matched microbiome and metabolome samples per subject/sample), so cross-validated metabolite prediction from taxa is not feasible.

## Inputs

- Microbial count CSV (samples by microbial features), e.g. data/IBD/microbiome_PRISM.csv
- Metabolite count CSV (samples by metabolomic features), e.g. data/IBD/metabolome_PRISM.csv
- External test-set microbial and metabolome CSVs (optional, for external validation, e.g. data/IBD/microbiome_external.csv, metabolome_external.csv)
- Metabolite annotation CSV (optional, e.g. data/IBD/metabolome_annotation.csv)
- Network hyper-parameter JSON file (number of layers, layer size, L2 penalty, dropout rate)

## Outputs

- Per-metabolite Spearman correlation coefficients (SCC) from cross-validated evaluation
- Shuffled-sample background SCC distribution and well-predicted metabolite list (SCC above 95th percentile of background)
- Mean SCC per dataset and well-predicted metabolite counts (e.g. 366 IBD, 143 CF, 29 soil)
- Trained MiMeNet models and output directory results

## How to apply

Load the paired sample-by-feature CSVs (e.g. microbiome_PRISM.csv and metabolome_PRISM.csv) plus, for IBD, the external_micro/external_metab test set and metabolome_annotation.csv. Preprocess by applying a compositional transformation (relative abundance, CLR, or 'None' if already transformed) and filtering low-abundant microbial and metabolite features. Tune the MLPNN hyper-parameters (number of layers, layer size, L2 penalty, dropout rate) into a network_parameters JSON/txt file, reusing shared hyper-parameters tuned from the first partition across all cross-validation iterations. Run MiMeNet_train.py with -num_run_cv 10 and -num_cv 10 (ten iterations of 10-fold cross-validation), then generate a shuffled-sample background SCC distribution (num_background >= 10 recommended) and flag metabolites whose SCC exceeds the 95th percentile of the background as well-predicted. Report mean SCC per dataset (restricted to annotated metabolites for IBD) and compare against reported values: 0.309 (IBD annotated), 0.457 (cystic fibrosis), 0.264 (soil), 0.275 (IBD external).

## Related tools

- **MiMeNet** (MLPNN framework trained with MiMeNet_train.py to predict metabolomic profiles from microbial features under 10 iterations of 10-fold cross-validation) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -external_micro data/IBD/microbiome_external.csv -external_metab data/IBD/metabolome_external.csv -micro_norm None -metab_norm CLR -net_params results/IBD/network_parameters.txt -annotation data/IBD/metabolome_annotation.csv -num_run_cv 10 -output IBD
```

## Evaluation signals

- Mean SCC within tolerance of reported values: 0.309 (IBD annotated), 0.457 (cystic fibrosis), 0.264 (soil), 0.275 (IBD external)
- Well-predicted metabolite counts of the same order as reported: 366 (IBD), 143 (CF), 29 (soil)
- SCC output exists per metabolite from the cross-validated evaluation of MiMeNet_train.py
- Every metabolite flagged well-predicted exceeds the 95th percentile of the shuffled-sample background SCC distribution
- Shared hyper-parameters (network_parameters file) tuned from the first partition are reused across all 10 cross-validation iterations

## Limitations

- Not all metabolites may be associated with microbes, so some metabolites have lower prediction correlations and the overall mean correlation across all metabolites is lower.
- A higher threshold value was observed for the soil data, possibly due to longitudinal observations (unresolved).
- MiMeNet, like compared methods, is still limited in providing biological plausibility and mechanistic insights.
- Requires Tensorflow 1.14 and Python 3.7+ per the README prerequisites.

## Evidence

- [intro] MiMeNet (Microbiome-Metabolome Network), a multi-layer perceptron neural network (MLPNN) that models the community metabolome profile using metagenomic taxonomic or functional features: "we present MiMeNet (Microbiome-Metabolome Network), a multi-layer perceptron neural network (MLPNN) that models the community metabolome profile using metagenomic taxonomic or functional features"
- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [methods] MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [supplementary] Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition: "Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition"
- [readme] MiMeNet requires Tensorflow 1.14 and is tested on Python 3.7+; example invocation with IBD PRISM data.: "python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -external_micro data/IBD/microbiome_external.csv -external_metab data/IBD/metabolome_external.csv"
