---
name: shuffled-background-thresholding
description: Use when cross-validated per-metabolite SCCs are available (e.g., from
  10 iterations of 10-fold CV with MiMeNet) and you need a statistically grounded
  count of well-predicted metabolites rather than a fixed correlation threshold.
when_to_use_negative:
- You need metabolite-level significance for a single model's associations (e.g.,
  microbe-metabolite correlation testing) rather than counting predictably well-modeled
  metabolites across CV folds.
- The metabolite set is so small or SCC distributions are so degenerate that a 95th-percentile
  empirical background is unstable (e.g., few-fold CV with very few metabolites).
- You are comparing methods using mean SCC alone (the article reports both mean SCC
  and well-predicted counts separately); the background threshold is only needed for
  the count metric.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiMeNet
  role: Generates the cross-validated per-metabolite SCCs and computes shuffled-sample
    backgrounds used to define the 95th-percentile well-predicted threshold
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Linear Elastic-Net baseline whose per-metabolite SCCs are evaluated against
    the same background-thresholding scheme for comparison
  repo: https://github.com/biobakery/melonnpan
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:273bb2fb31d954a1c39932dec5b5e6708d557fbad4fdb45e0386f35f2eda1af8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# shuffled-background-thresholding

## Summary

Defines 'well-predicted' metabolites by comparing each metabolite's cross-validated Spearman correlation coefficient (SCC) against an empirical background of SCCs from shuffled samples, flagging metabolites whose SCC exceeds the 95th percentile of that background. This avoids arbitrary correlation cutoffs when counting how many metabolites the MiMeNet model predicts well.

## When to use

Use when cross-validated per-metabolite SCCs are available (e.g., from 10 iterations of 10-fold CV with MiMeNet) and you need a statistically grounded count of well-predicted metabolites rather than a fixed correlation threshold.

## When NOT to use

- You need metabolite-level significance for a single model's associations (e.g., microbe-metabolite correlation testing) rather than counting predictably well-modeled metabolites across CV folds.
- The metabolite set is so small or SCC distributions are so degenerate that a 95th-percentile empirical background is unstable (e.g., few-fold CV with very few metabolites).
- You are comparing methods using mean SCC alone (the article reports both mean SCC and well-predicted counts separately); the background threshold is only needed for the count metric.

## Inputs

- Cross-validated per-metabolite SCC values from MiMeNet (samples-by-metabolite prediction outputs over 10x10-fold CV)
- Sample-by-microbial-feature matrix (e.g., data/IBD/microbiome_PRISM.csv)
- Sample-by-metabolite matrix (e.g., data/IBD/metabolome_PRISM.csv)
- Previously generated background directory (optional, passed via -background)
- Network hyperparameters JSON (e.g., results/IBD/network_parameters.txt)

## Outputs

- List of well-predicted metabolites (SCC above the 95th percentile of the shuffled background)
- Per-metabolite 95th-percentile SCC thresholds from the empirical background
- Well-predicted metabolite counts written to the comparison output (e.g., annotated_vs_full_scc_comparison.csv)

## How to apply

After MiMeNet evaluates models with 10 iterations of 10-fold cross-validation (num_cv=10) producing per-metabolite SCCs, generate an empirical null distribution by re-running the cross-validation pipeline on shuffled sample labels (the background), controlled by the -num_background parameter (use num_background >= 10 background CV iterations per the ablation protocol). For each metabolite, compute the 95th percentile of its shuffled-sample SCC background; any metabolite whose observed SCC exceeds this per-metabolite 95th percentile threshold is called well-predicted. This per-metabolite empirical thresholding is used in the IBD (PRISM) ablation where full-metabolite training raised the well-predicted annotated-metabolite count from 333 to 366, and in the main comparisons where MiMeNet increased well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29 across three datasets. Decision point: use the -background flag to reuse a previously generated background directory rather than recomputing it.

## Related tools

- **MiMeNet** (Generates the cross-validated per-metabolite SCCs and computes shuffled-sample backgrounds used to define the 95th-percentile well-predicted threshold) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Linear Elastic-Net baseline whose per-metabolite SCCs are evaluated against the same background-thresholding scheme for comparison) — https://github.com/biobakery/melonnpan

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -output results/IBD -net_params results/IBD/network_parameters.txt -micro_norm None -metab_norm CLR -num_run_cv 10 -num_cv 10 -num_background 10
```

## Evaluation signals

- Well-predicted metabolites must be those whose SCC exceeds the 95th percentile of their shuffled-sample empirical background with num_background >= 10 iterations.
- On the IBD (PRISM) annotated-only model, the thresholding must recover 333 well-predicted annotated metabolites; with full-metabolome training it must recover 366.
- In the main IBD comparison, MiMeNet's thresholding should yield 366 well-predicted metabolites versus 198 for the linear baseline.
- The -num_background parameter must be set (background CV iterations actually executed or loaded via -background), not left at an unexamined default.
- Well-predicted counts should track mean SCC directionality (higher mean SCC implies equal or higher count) as seen in the reported 0.259 to 0.309 annotated-subset SCC increase.

## Limitations

- The threshold is empirical and dataset-dependent; a higher threshold value was observed for the soil dataset, possibly due to longitudinal observations (unresolved in the paper).
- Not all metabolites are microbe-associated, so some metabolites will have low SCCs and never pass the 95th-percentile threshold, lowering overall counts regardless of model quality.
- Computing the background requires additional CV iterations (num_background >= 10), adding runtime cost unless a previously generated background is reused.

## Evidence

- [methods] Per-metabolite 95th-percentile rule: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [readme] num_background CLI parameter: "-num_background NUM_BACKGROUND    Number of background CV Iterations"
- [abstract] Well-predicted counts from the abstract: "identifies more well-predicted metabolites (increase in the number of well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29)"
