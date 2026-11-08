---
name: spearman-correlation-threshold-derivation
description: Use when you need an objective, dataset-specific significance threshold
  for per-metabolite prediction correlations in a paired microbiome-metabolome regression
  task, rather than an arbitrary fixed SCC cutoff.
when_to_use_negative:
- When an external validated significance threshold already exists for the dataset
  — the derived cutoff is dataset-specific (0.129 to 0.410 across MiMeNet's three
  datasets) and does not transfer between datasets
- When the goal is feature selection or interaction inference rather than flagging
  well-predicted metabolites — use MiMeNet's module construction or interaction-score
  workflow instead
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: MiMeNet
  role: MLPNN framework that trains shuffled-background and real cross-validated models,
    computes per-metabolite SCCs, and thresholds at the 95th percentile
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:68291b7941b573af20e6b57ad55a0969b82da120696147d78cdcc35118c121f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spearman-correlation-threshold-derivation

## Summary

Derives dataset-specific Spearman correlation coefficient (SCC) significance cutoffs by building an empirical null distribution of per-metabolite SCCs from shuffled-label cross-validated runs, then calling metabolites whose real SCC exceeds the 95th percentile as well-predicted.

## When to use

Use when you need an objective, dataset-specific significance threshold for per-metabolite prediction correlations in a paired microbiome-metabolome regression task, rather than an arbitrary fixed SCC cutoff.

## When NOT to use

- When an external validated significance threshold already exists for the dataset — the derived cutoff is dataset-specific (0.129 to 0.410 across MiMeNet's three datasets) and does not transfer between datasets
- When the goal is feature selection or interaction inference rather than flagging well-predicted metabolites — use MiMeNet's module construction or interaction-score workflow instead

## Inputs

- Paired sample-by-microbe count matrix (comma-delimited CSV)
- Paired sample-by-metabolite count matrix (comma-delimited CSV)
- Network hyperparameters JSON file
- Normalization choice per matrix: RA, CLR, or None

## Outputs

- Empirical background SCC distribution per dataset (from shuffled-label cross-validated runs)
- Dataset-specific 95th-percentile SCC significance cutoff (e.g. 0.136 IBD, 0.129 CF, 0.410 soil)
- List of well-predicted metabolites per dataset (SCC above cutoff), e.g. 366, 143, and 29 metabolites

## How to apply

Load the paired microbial and metabolite count tables (e.g. the IBD/PRISM, cystic fibrosis, and soil wetting datasets), apply MiMeNet's compositional transformations (relative abundance or center log-ratio) via -micro_norm and -metab_norm, and filter low-abundant features. Run the background procedure: for at least 10 iterations of 10-fold cross-validation (-num_background), randomly shuffle sample labels each iteration and train the MLPNN models using the provided network hyperparameters JSON (-net_params). Pool the per-metabolite SCC values across all shuffled iterations to build each dataset's empirical background SCC distribution, and take the 95th percentile as the significance cutoff. Compare each metabolite's SCC from the unshuffled cross-validated evaluation against this cutoff; metabolites above it are well-predicted. This produced cutoffs of 0.136 (IBD/PRISM), 0.129 (cystic fibrosis), and 0.410 (soil), yielding well-predicted metabolite counts of 366, 143, and 29 respectively.

## Related tools

- **MiMeNet** (MLPNN framework that trains shuffled-background and real cross-validated models, computes per-metabolite SCCs, and thresholds at the 95th percentile) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -output out/ -net_params net_params.json -micro_norm RA -metab_norm RA -num_background 10 -num_run_cv 10 -num_cv 10
```

## Evaluation signals

- Derived 95th-percentile cutoffs should approximate the published values: 0.136 (IBD/PRISM), 0.129 (cystic fibrosis), 0.410 (soil)
- Well-predicted metabolite counts should approximate 366, 143, and 29 across the three datasets, within stochastic variation of the shuffled background runs
- Background SCC distribution must be built from at least 10 iterations of 10-fold cross-validation with sample labels shuffled each iteration
- Per-metabolite SCC values from the unshuffled cross-validated evaluation must be compared against the same dataset's own cutoff, not a shared threshold

## Limitations

- The soil dataset showed a notably higher threshold (0.410), which the authors attribute possibly to longitudinal observations and leave unresolved
- Because not all metabolites are associated with microbes, some metabolites will have low prediction correlations regardless of model quality, lowering the overall mean SCC
- Cutoffs are stochastic — they depend on the random shuffles in the background runs, so exact values vary between runs

## Evidence

- [methods] MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [methods] Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [discussion] We also observed a higher threshold value for the soil data (Fig 3A–3C), which may be due to the longitudinal observations.: "We also observed a higher threshold value for the soil data (Fig 3A–3C), which may be due to the longitudinal observations."
- [readme] -num_background NUM_BACKGROUND    Number of background CV Iterations: "-num_background NUM_BACKGROUND    Number of background CV Iterations"
