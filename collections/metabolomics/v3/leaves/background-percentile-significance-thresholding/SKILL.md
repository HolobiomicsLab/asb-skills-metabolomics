---
name: background-percentile-significance-thresholding
description: Use when you need to decide which metabolites or microbe-metabolite interactions
  are statistically significant after cross-validated prediction on paired microbiome-metabolome
  data, rather than picking arbitrary cutoffs.
when_to_use_negative:
- Your data is not paired microbiome-metabolome samples, so there is no cross-validated
  prediction framework from which to build a background distribution
- You already have an established empirical null (e.g., a matched control cohort)
  and do not need to generate a shuffled-data background via cross-validation
- You need significance for a single linear model (e.g., MelonnPan's Elastic Net)
  rather than weights aggregated across 100 neural-network cross-validation models
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3452
tools:
- name: MiMeNet
  role: Performs the cross-validated training and provides the -background and -threshold
    CLI parameters used to establish background iterations and significance cutoffs
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Compared linear baseline that uses a different significance regime; not used
    for generating the background itself
  repo: https://github.com/biobakery/melonnpan
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:0cc540cf4f61de2aa67bd1662339f6fde1053c1343e05e16f3cb08c919ee3366
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# background-percentile-significance-thresholding

## Summary

Establishes an empirical background distribution from cross-validated iterations on shuffled data and applies percentile-based cutoffs (e.g., 95th for well-predicted metabolites, 97.5th for significant interactions) to MiMeNet's attribution-score matrix.

## When to use

Use when you need to decide which metabolites or microbe-metabolite interactions are statistically significant after cross-validated prediction on paired microbiome-metabolome data, rather than picking arbitrary cutoffs.

## When NOT to use

- Your data is not paired microbiome-metabolome samples, so there is no cross-validated prediction framework from which to build a background distribution
- You already have an established empirical null (e.g., a matched control cohort) and do not need to generate a shuffled-data background via cross-validation
- You need significance for a single linear model (e.g., MelonnPan's Elastic Net) rather than weights aggregated across 100 neural-network cross-validation models

## Inputs

- Microbe-by-sample comma-delimited feature matrix (micro) and metabolite-by-sample matrix (metab) from the paired dataset (e.g., IBD PRISM)
- Background directory with previously generated cross-validated background iterations (shuffled-data models)
- Trained MiMeNet model weights from 10 iterations of 10-fold cross-validation (100 models) on real data

## Outputs

- Set of well-predicted metabolites (SCC above the 95th percentile of the background)
- Set of significant microbe-metabolite interactions (passing the 97.5th-percentile background cutoff)
- Filtered interaction-score matrix (e.g., 163 significant microbes x 6857 well-predicted metabolites) written to interaction_scores.csv

## How to apply

Run the full cross-validated pipeline (10 iterations of 10-fold cross-validation in MiMeNet, yielding 100 models) twice: once on real paired data and once on shuffled data to build a null/background distribution. Compute the per-metabolite Spearman correlation coefficients (SCC) and take metabolites whose SCC exceeds the 95th percentile of the background as well-predicted; the paper's MiMeNet -threshold parameter can also set an explicit significance cutoff. For interaction-level attribution, multiply consecutive hidden-layer weight matrices per model, average the 100 per-model microbe-by-metabolite matrices, and compare each element against the 97.5th-percentile cutoff of the background distribution to call a microbe-metabolite interaction significant. Rationale: the shuffled-data background accounts for the high-dimensional overfitting that would otherwise inflate apparent microbe-metabolite associations.

## Related tools

- **MiMeNet** (Performs the cross-validated training and provides the -background and -threshold CLI parameters used to establish background iterations and significance cutoffs) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Compared linear baseline that uses a different significance regime; not used for generating the background itself) — https://github.com/biobakery/melonnpan

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -background background_dir -num_background 100 -threshold 97.5 -num_run_cv 10 -num_cv 10 -output out/
```

## Evaluation signals

- The background must come from shuffled-data cross-validated iterations (same 10 iterations of 10-fold CV structure as the real-data run), not from a simple permutation test on a single model
- Well-predicted metabolites must be defined relative to the 95th percentile of the SCC distribution — the IBD (PRISM) run yields 6857 well-predicted metabolites
- Significant microbe-metabolite interactions must pass the 97.5th-percentile cutoff of the background; on IBD (PRISM) this retains 163 microbes with at least one significant interaction
- The final output matrix dimensions must match the retained significant microbes x well-predicted metabolites (163 x 6857 for IBD PRISM) and be written as interaction_scores.csv
- The -threshold CLI parameter (if set) should be consistent with the reported significance cutoff rather than an arbitrary fixed value

## Limitations

- The background cutoff depends on the number of background CV iterations (-num_background); too few iterations makes the percentile estimate noisy
- Higher threshold values were observed for the soil dataset, possibly due to longitudinal observations, indicating that the same percentile cutoff is not transferable across datasets
- Since not all metabolites may be associated with microbes, some metabolites will have lower prediction correlations, so the 95th-percentile cutoff is dataset-specific

## Evidence

- [methods] MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [other] Element-wise average the 100 per-model attribution matrices to obtain the mean microbe-metabolite interaction-score matrix.: [evidence span withheld — anchor status: not_found]
- [readme] -threshold THRESHOLD Define significant correlation threshold: "-threshold THRESHOLD Define significant correlation threshold"
- [discussion] We also observed a higher threshold value for the soil data (Fig 3A–3C), which may be due to the longitudinal observations.: "We also observed a higher threshold value for the soil data (Fig 3A–3C), which may be due to the longitudinal observations."
