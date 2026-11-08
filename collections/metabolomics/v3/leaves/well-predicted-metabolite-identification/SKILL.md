---
name: well-predicted-metabolite-identification
description: Use when you have paired microbial feature tables (RA or CLR transformed)
  and metabolite tables and need to flag which metabolites a MiMeNet model predicts
  significantly better than chance, rather than reporting only mean SCC.
when_to_use_negative:
- You only need the mean prediction SCC across all metabolites — the mean is reported
  directly and does not require the shuffled background cutoff procedure.
- The metabolites' prediction correlations are already being thresholded by a user-supplied
  fixed value via -threshold; use that path instead of deriving a new background percentile.
- No paired (matched samples) microbial-metabolite data exist — the background shuffle-and-compare
  design requires paired samples per dataset.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: MiMeNet
  role: Trains the MLPNN on shuffled and unshuffled paired microbiome-metabolite data
    via 10-iteration 10-fold CV to generate background and observed per-metabolite
    SCC distributions for cutoff derivation
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:4c7f388d89c2deeabbe1156dec086081ef4425062013d8ae24fc7baaf31f6111
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# well-predicted-metabolite-identification

## Summary

Identify which metabolites a microbiome-metabolome prediction model (MiMeNet MLPNN) predicts significantly well by comparing each metabolite's cross-validated Spearman correlation coefficient (SCC) against a dataset-specific 95th-percentile cutoff derived from a shuffled-label background distribution. This converts raw per-metabolite SCCs into a principled count of well-predicted metabolites per dataset.

## When to use

Use when you have paired microbial feature tables (RA or CLR transformed) and metabolite tables and need to flag which metabolites a MiMeNet model predicts significantly better than chance, rather than reporting only mean SCC.

## When NOT to use

- You only need the mean prediction SCC across all metabolites — the mean is reported directly and does not require the shuffled background cutoff procedure.
- The metabolites' prediction correlations are already being thresholded by a user-supplied fixed value via -threshold; use that path instead of deriving a new background percentile.
- No paired (matched samples) microbial-metabolite data exist — the background shuffle-and-compare design requires paired samples per dataset.

## Inputs

- Comma delimited CSV matrix of samples by microbial features (-micro)
- Comma delimited CSV matrix of samples by metabolomic features (-metab)
- JSON file of network hyperparameters (-net_params)
- Normalization choice for each omics layer: RA, CLR, or None (-micro_norm, -metab_norm)
- Number of background CV iterations with shuffled labels (-num_background)

## Outputs

- Per-metabolite SCC evaluation values from shuffled background CV runs (empirical background distribution)
- Dataset-specific 95th-percentile SCC significance cutoff (e.g. 0.136 IBD, 0.129 CF, 0.410 soil)
- List/tally of well-predicted metabolites per dataset (e.g. well_predicted_metabolites.csv with counts such as 366, 143, 29)

## How to apply

Load the paired samples-by-features CSV tables and run MiMeNet's background procedure: for at least 10 iterations of 10-fold cross-validation (num_background parameter), randomly shuffle sample labels each iteration and train the MLPNN models with the network hyperparameters JSON (net_params), applying the same compositional transformation (relative abundance or center log-ratio) used for the real run. Collect the per-metabolite SCC evaluation values from all shuffled cross-validated iterations to build an empirical background SCC distribution per dataset. Compute the 95th-percentile SCC of that background distribution as the dataset-specific significance cutoff — the paper derives 0.136 for IBD (PRISM), 0.129 for cystic fibrosis, and 0.410 for soil. Compare each metabolite's SCC from the unshuffled cross-validated evaluation against the cutoff; metabolites exceeding it are marked well-predicted and tallied. The paper reports MiMeNet well-predicted counts of 366, 143, and 29 across the IBD, cystic fibrosis, and soil datasets (vs 198, 104, and 4 for the linear Elastic Net baseline), so your counts should fall near these values within the stochastic variation of the shuffled background runs.

## Related tools

- **MiMeNet** (Trains the MLPNN on shuffled and unshuffled paired microbiome-metabolite data via 10-iteration 10-fold CV to generate background and observed per-metabolite SCC distributions for cutoff derivation) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -output out/ -net_params net_params.json -num_background 10 -micro_norm RA -metab_norm RA -num_run_cv 10 -num_cv 10
```

## Evaluation signals

- The 95th-percentile cutoffs should be dataset-specific and near the reported values: 0.136 (IBD/PRISM), 0.129 (cystic fibrosis), 0.410 (soil), within stochastic variation of the shuffled background runs.
- Well-predicted metabolite counts should be near the paper's MiMeNet values of 366 (IBD), 143 (cystic fibrosis), and 29 (soil).
- The background (shuffled-label) SCC distribution must be built from ≥10 iterations of 10-fold CV; check the num_background setting matches the number of collected per-metabolite SCC values.
- Counts of well-predicted metabolites should exceed the linear baseline (MelonnPan Elastic Net) counts of 198, 104, and 4 for the corresponding datasets when MiMeNet is used.
- The soil dataset's cutoff is expected to be markedly higher than the host-associated datasets; a cutoff near 0.13 for soil signals a pipeline error.

## Limitations

- The shuffled background procedure is stochastic, so derived cutoffs and counts vary between runs; results are only reproducible within expected stochastic variation.
- A higher threshold value was observed for the soil data, which may be due to the longitudinal observations — this remains unresolved in the paper.
- Since not all metabolites may be associated with microbes, some metabolites will have lower prediction correlations, resulting in lower mean correlation across all metabolites regardless of cutoff choice.
- The resulting well-predicted lists describe statistical predictability, not biological plausibility or mechanistic insight.

## Evidence

- [methods] MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [methods] Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [abstract] identifies more well-predicted metabolites (increase in the number of well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29): "identifies more well-predicted metabolites (increase in the number of well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29)"
- [discussion] We also observed a higher threshold value for the soil data (Fig 3A–3C), which may be due to the longitudinal observations.: "We also observed a higher threshold value for the soil data (Fig 3A–3C), which may be due to the longitudinal observations."
- [readme] -num_background NUM_BACKGROUND    Number of background CV Iterations: "-num_background NUM_BACKGROUND    Number of background CV Iterations"
