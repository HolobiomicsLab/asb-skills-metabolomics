---
name: neural-network-weight-matrix-attribution
description: Use when a trained MiMeNet multi-layer perceptron model set is available
  and you need to attribute microbe-metabolite interaction scores from network weights,
  e.g. to identify significant microbes (like the 163 significant microbes on the
  IBD PRISM dataset) tied to well-predicted metabolites.
when_to_use_negative:
- When the goal is simply to predict metabolite abundances from microbial features
  — use MiMeNet's standard prediction path instead, since this skill only extracts
  interaction attribution, not predictions.
- When the model is a linear predictor (e.g. MelonnPan's Elastic Net) rather than
  MiMeNet's MLPNN — there are no layered weight matrices to multiply, so this attribution
  approach does not apply.
- When only a single trained model is available without the 100-model cross-validated
  ensemble and its shuffled background distribution — significance cutoffs cannot
  be derived.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3577
tools:
- name: MiMeNet
  role: Trains the cross-validated MLPNN models whose weight matrices are multiplied
    and averaged to produce the microbe-metabolite attribution matrix; also supplies
    the shuffled-data background distribution for significance thresholds.
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: 'Comparison baseline: a regularized linear regression model whose coefficients
    cannot be composed the same way as MiMeNet''s MLPNN weights, motivating the network-weight
    attribution approach.'
  repo: https://github.com/biobakery/melonnpan
- name: BiomeNED
  role: 'Comparison baseline: an encoder-decoder model whose latent-space representation
    makes microbe-metabolite interaction interpretation less intuitive than direct
    weight-matrix attribution.'
  repo: https://github.com/vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:d64848341f2cc25402b52cdcdc220cbe076727236ba3214353c319bb57e61288
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - BiomeNED
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# neural-network-weight-matrix-attribution

## Summary

Derives a microbe-by-metabolite interaction-score matrix from the learned weight matrices of MiMeNet's cross-validated MLPNN models (10 iterations of 10-fold CV = 100 models), by multiplying consecutive hidden-layer weight matrices and averaging element-wise across all 100 models.

## When to use

Use when a trained MiMeNet multi-layer perceptron model set is available and you need to attribute microbe-metabolite interaction scores from network weights, e.g. to identify significant microbes (like the 163 significant microbes on the IBD PRISM dataset) tied to well-predicted metabolites.

## When NOT to use

- When the goal is simply to predict metabolite abundances from microbial features — use MiMeNet's standard prediction path instead, since this skill only extracts interaction attribution, not predictions.
- When the model is a linear predictor (e.g. MelonnPan's Elastic Net) rather than MiMeNet's MLPNN — there are no layered weight matrices to multiply, so this attribution approach does not apply.
- When only a single trained model is available without the 100-model cross-validated ensemble and its shuffled background distribution — significance cutoffs cannot be derived.

## Inputs

- Trained MLPNN weight matrices from 10 iterations of 10-fold cross-validated MiMeNet models (100 models total)
- Empirical background distribution from shuffled-data cross-validated iterations (for 97.5th-percentile significance cutoff)
- Metabolite SCC evaluation values from cross-validated prediction (to select well-predicted metabolites above the 95th percentile)

## Outputs

- Mean microbe-by-metabolite interaction-score matrix averaged across all 100 cross-validated models
- Filtered interaction_scores.csv containing significant microbe rows and well-predicted metabolite columns (e.g. 163 microbes x 6857 metabolites for IBD PRISM)

## How to apply

Load the trained MLPNN weight matrices from the 10 iterations of 10-fold cross-validation (100 models total) using MiMeNet utilities. For each model, multiply its consecutive hidden-layer weight matrices to yield a per-model microbe-by-metabolite attribution matrix. Element-wise average the 100 per-model matrices into a single mean interaction-score matrix. Load the empirical background distribution from shuffled-data cross-validated iterations and compute the 97.5th-percentile cutoff for microbe-metabolite interactions; retain only rows (microbes) passing this threshold. Subset columns to the well-predicted metabolites — those with Spearman correlation coefficient (SCC) above the 95th percentile of the background — then write the filtered matrix (e.g. 163 microbes x 6857 metabolites on the IBD PRISM data) to interaction_scores.csv.

## Related tools

- **MiMeNet** (Trains the cross-validated MLPNN models whose weight matrices are multiplied and averaged to produce the microbe-metabolite attribution matrix; also supplies the shuffled-data background distribution for significance thresholds.) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Comparison baseline: a regularized linear regression model whose coefficients cannot be composed the same way as MiMeNet's MLPNN weights, motivating the network-weight attribution approach.) — https://github.com/biobakery/melonnpan
- **BiomeNED** (Comparison baseline: an encoder-decoder model whose latent-space representation makes microbe-metabolite interaction interpretation less intuitive than direct weight-matrix attribution.) — https://github.com/vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro microbes.csv -metab metabolites.csv -output out_dir -num_run_cv 10 -num_cv 10 -background background_dir -num_background 100 -micro_norm RA -metab_norm RA
```

## Evaluation signals

- The output matrix must have exactly 163 significant microbe rows and 6857 well-predicted metabolite columns when applied to the IBD (PRISM) dataset.
- The mean interaction-score matrix must be the element-wise average of exactly 100 per-model attribution matrices (10 iterations of 10-fold cross-validation).
- Every retained microbe row must have at least one interaction score exceeding the 97.5th-percentile cutoff computed from the shuffled-data background distribution.
- Retained metabolite columns must correspond to metabolites whose cross-validated SCC is above the 95th percentile of the background (well-predicted metabolites).
- The final artifact must be a single CSV (interaction_scores.csv) whose cell values are averaged interaction scores, not per-model raw weights.

## Limitations

- Attribution relies on an empirical background from shuffled-data cross-validation; without it, the 97.5th-percentile microbe-metabolite significance threshold cannot be defined.
- Some metabolites will have lower prediction correlations because not all metabolites may be associated with microbes; only the well-predicted subset is retained, so the matrix cannot cover the full metabolome.
- The method is limited in providing biological plausibility and mechanistic insights — attribution scores do not prove causation.
- Procedures to discern statistical association of microbe-metabolite interactions to host diseases do not yet exist as a validated step.

## Evidence

- [abstract] Using ten iterations of 10-fold cross-validation on three paired microbiome-metabolome datasets: "Using ten iterations of 10-fold cross-validation on three paired microbiome-metabolome datasets"
- [methods] This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns.: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [methods] MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [supplementary] Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition: "Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition"
- [readme] Comma delimited file representing matrix of samples by microbial features: "Comma delimited file representing matrix of samples by microbial features"
- [results] Using the attribution score matrix computed from the learned cross-validated network weights on the IBD (PRISM) dataset, MiMeNet identified 163 microbes that had at least one significant attribution score with a well-predicted metabolite.: [evidence span withheld — anchor status: not_found]
