---
name: wilcoxon-signed-rank-paired-testing
description: Use when two models are evaluated on the SAME set of features (e.g. the
  466 annotated metabolites of the IBD PRISM dataset) under paired CV settings, and
  you need a nonparametric test on paired per-feature SCC differences.
when_to_use_negative:
- The two SCC distributions come from independent, unpaired evaluations (e.g. different
  metabolite sets or different CV folds), since the signed-rank test requires metabolite-level
  pairing
- Comparing more than two models simultaneously, where a Friedman test would be the
  appropriate paired nonparametric alternative
- Metabolites with zero SCC difference dominate the pairs, invalidating rank-based
  inference on the signed differences
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MiMeNet
  role: Produces the paired per-metabolite SCC distributions (full-metabolome vs annotated-only
    models under 10x10-fold CV) that the Wilcoxon signed-rank test is applied to
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:5e05acaadbc10bb52d426addda0482e68c902a1db2edcc057a79cc3394ac81cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Wilcoxon signed-rank paired testing

## Summary

Nonparametric paired significance test applied to per-metabolite Spearman correlation coefficients (SCC) from two MiMeNet models trained under identical cross-validation, to determine whether full-metabolome multivariate learning significantly improves prediction of the annotated metabolite subset.

## When to use

Use when two models are evaluated on the SAME set of features (e.g. the 466 annotated metabolites of the IBD PRISM dataset) under paired CV settings, and you need a nonparametric test on paired per-feature SCC differences.

## When NOT to use

- The two SCC distributions come from independent, unpaired evaluations (e.g. different metabolite sets or different CV folds), since the signed-rank test requires metabolite-level pairing
- Comparing more than two models simultaneously, where a Friedman test would be the appropriate paired nonparametric alternative
- Metabolites with zero SCC difference dominate the pairs, invalidating rank-based inference on the signed differences

## Inputs

- Per-metabolite SCC vector from the full-metabolome MiMeNet model (annotated subset)
- Per-metabolite SCC vector from the annotated-only MiMeNet model, paired by metabolite ID
- data/IBD/metabolome_annotation.csv defining the 466-annotated-metabolite pairing universe
- annotated_vs_full_scc_comparison.csv target output table

## Outputs

- Wilcoxon signed-rank test statistic and two-sided p-value on paired annotated-subset SCCs
- annotated_vs_full_scc_comparison.csv containing per-metabolite paired SCCs and well-predicted counts (333 vs 366)

## How to apply

First generate two paired vectors of per-metabolite SCCs by running MiMeNet twice with identical settings (10 iterations of 10-fold cross-validation, hyperparameters from results/IBD/network_parameters.txt): once on the full 8848-metabolite output layer and once restricted to the 466 annotated metabolites from data/IBD/metabolome_annotation.csv. Pair the SCCs by metabolite identity (the same metabolite in both models) so the signed differences reflect model choice, not sampling variance. Apply the Wilcoxon signed-rank test to these paired differences with scipy.stats.wilcoxon; the signed-rank test is appropriate because per-metabolite SCC differences need not be normally distributed and only paired ranks are used. For the IBD (PRISM) ablation, the expected result is a highly significant elevation of annotated-subset SCCs (mean 0.259 annotated-only vs 0.309 full-metabolome) supporting the 'guilt by association' transfer-learning rationale of multivariate output layers. Write the test outcome alongside the paired SCCs and well-predicted counts (SCC above the 95th-percentile shuffled-sample background threshold) to annotated_vs_full_scc_comparison.csv.

## Related tools

- **MiMeNet** (Produces the paired per-metabolite SCC distributions (full-metabolome vs annotated-only models under 10x10-fold CV) that the Wilcoxon signed-rank test is applied to) — https://github.com/YDaiLab/MiMeNet

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(scc_full_metabolome, scc_annotated_only)
```

## Evaluation signals

- Wilcoxon signed-rank P < 10^-47 for the paired annotated-subset SCCs (full vs annotated-only on IBD PRISM)
- Annotated-subset mean SCC increases from 0.259 (annotated-only) to 0.309 (full-metabolome model)
- Well-predicted annotated metabolite count (SCC above 95th-percentile shuffled background, num_background >= 10) increases from 333 to 366
- Pairing verified: both SCC vectors are indexed over the same 466 annotated metabolites, not over the full 8848-metabolite output layer

## Limitations

- The test only certifies statistical significance of the paired SCC difference; the article notes that not all metabolites may be microbe-associated, so some metabolites will have low SCC regardless of model
- Results are dataset-specific — the IBD (PRISM) numbers (0.259 vs 0.309, P < 10^-47) do not transfer to the cystic fibrosis or soil wetting datasets without re-running the paired CV comparison

## Evidence

- [discussion] annotated benefited from multivariate learning: "the MiMeNet prediction on the set of the annotated metabolites benefited from including tasks of predicting the rest of the unannotated metabolites"
- [methods] paired SCC generation basis: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [methods] well-predicted threshold definition: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [readme] CV pairing control via MiMeNet flags: "-num_run_cv NUM_RUN_CV            Number of iterations for"
