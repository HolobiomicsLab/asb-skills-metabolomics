---
name: cross-validated-model-evaluation
description: Use when you have paired microbial-feature and metabolite-count matrices
  for the same samples and need an honest estimate of how well a predictive model
  recovers each metabolite across held-out folds.
when_to_use_negative:
- You only have a single metabolite table and no paired microbial features — cross-validated
  SCC is undefined without a predictor–target pair.
- Your goal is to rank microbe–metabolite interactions rather than evaluate overall
  predictive accuracy; use the interaction-score/attribution workflow instead.
- The datasets are tiny and repeated 10-fold CV would leave too few samples per fold
  for the MLPNN to train reliably.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: MiMeNet
  role: 'Primary framework: trains the MLPNN and runs the repeated k-fold cross-validation
    that produces the per-metabolite SCCs.'
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Baseline linear-regression predictor used as the comparison model whose mean
    SCC and well-predicted metabolite counts MiMeNet is evaluated against.
  repo: https://github.com/biobakery/melonnpan
- name: BiomeNED
  role: Alternative neural encoder–decoder baseline (NED) compared against MiMeNet
    on external IBD validation using cross-validated evaluation.
  repo: https://github.com/vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:78157a9f8dc5eb2042870ca373038cbfa820d19e66cb1e02821ee7f1b35910ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - BiomeNED
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-validated-model-evaluation

## Summary

Cross-validated evaluation of microbiome-to-metabolite predictors (e.g. MiMeNet) by running repeated k-fold CV, computing per-metabolite Spearman correlation coefficients (SCC), and comparing against baseline linear models such as MelonnPan on the same datasets.

## When to use

Use when you have paired microbial-feature and metabolite-count matrices for the same samples and need an honest estimate of how well a predictive model recovers each metabolite across held-out folds.

## When NOT to use

- You only have a single metabolite table and no paired microbial features — cross-validated SCC is undefined without a predictor–target pair.
- Your goal is to rank microbe–metabolite interactions rather than evaluate overall predictive accuracy; use the interaction-score/attribution workflow instead.
- The datasets are tiny and repeated 10-fold CV would leave too few samples per fold for the MLPNN to train reliably.

## Inputs

- Comma-delimited samples-by-microbial-features matrix (-micro)
- Comma-delimited samples-by-metabolomic-features matrix (-metab)
- Optional external validation matrices (-external_micro, -external_metab)
- Optional metabolite annotation file (-annotation)
- Optional JSON file of network hyper-parameters (-net_params)
- Normalization choice for microbiome and metabolome (RA, CLR, or None)

## Outputs

- Per-metabolite Spearman correlation coefficient (SCC) values from cross-validation
- Set of well-predicted metabolites (SCC above the 95th percentile of the background)
- Mean SCC comparison table vs. baseline models (e.g. MelonnPan)
- Robustness curves of mean SCC across k=10, 5, 3, 2 folds and 100%/80%/60%/40% sample subsets

## How to apply

Load the paired samples-by-microbes and samples-by-metabolites comma-delimited matrices into MiMeNet, applying a compositional transformation (RA or CLR, or 'None' if already transformed) and filtering low-abundant features. Run MiMeNet_train.py with -num_cv set to the fold count (10, 5, 3, or 2) and -num_run_cv set to the number of CV iterations (default 10), so the model is retrained and evaluated on held-out partitions each iteration. For each metabolite, compute the Spearman correlation coefficient (SCC) between its true and predicted values across all held-out predictions, then flag a metabolite as well-predicted if its SCC exceeds the 95th percentile of the background distribution (or set -threshold explicitly). Compare mean SCC against baselines like MelonnPan on the same datasets — e.g. MiMeNet raised mean SCC from 0.108 to 0.309 on the IBD (PRISM) dataset. Repeat under shared vs. per-partition hyper-parameter tuning and random sample subsetting (100%/80%/60%/40%) to test robustness.

## Related tools

- **MiMeNet** (Primary framework: trains the MLPNN and runs the repeated k-fold cross-validation that produces the per-metabolite SCCs.) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Baseline linear-regression predictor used as the comparison model whose mean SCC and well-predicted metabolite counts MiMeNet is evaluated against.) — https://github.com/biobakery/melonnpan
- **BiomeNED** (Alternative neural encoder–decoder baseline (NED) compared against MiMeNet on external IBD validation using cross-validated evaluation.) — https://github.com/vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -output out_dir -micro_norm RA -metab_norm RA -num_run_cv 10 -num_cv 10 -threshold 0.05
```

## Evaluation signals

- Each metabolite has an SCC value after every CV iteration; verify no metabolite is missing an SCC in the output table.
- A metabolite is labelled well-predicted only if its SCC exceeds the 95th percentile of the background distribution (or the explicit -threshold).
- Mean SCC should improve over the MelonnPan baseline by roughly 0.2 on the IBD (PRISM) dataset (0.108 → 0.309) and similarly on cystic fibrosis (0.276 → 0.457).
- Reducing folds from 10 to 2 or subsetting samples to 40% should degrade mean SCC only modestly (e.g. IBD PRISM: 0.297 → 0.218 at k=2, 0.287 → 0.179 at 40% subset).
- External validation SCC on the IBD (External) dataset should remain positive and comparable to cross-validated SCC, confirming the model generalises.

## Limitations

- Not all metabolites may be associated with microbes, so some will have low prediction correlations, lowering the overall mean correlation.
- MiMeNet, like MelonnPan and NED, is limited in providing biological plausibility and mechanistic insight beyond statistical association.
- A higher significance threshold was observed for the soil dataset, possibly due to its longitudinal structure.
- Robustness was tested under shared vs. per-partition hyper-parameter tuning; sharing hyper-parameters tuned from the first partition did not cause overfitting but is still a design choice that must be reported.

## Evidence

- [readme] Cross-validated SCC is the primary evaluation metric; folds and iterations are set via CLI flags.: "-num_run_cv NUM_RUN_CV            Number of iterations for"
- [abstract] MiMeNet more accurately predicts metabolite abundances than state-of-the-art linear models, with mean SCC gains reported across three paired datasets.: "MiMeNet more accurately predicts metabolite abundances (mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264)"
- [methods] The cross-validated evaluation pipeline and the SCC cutoff for well-predicted metabolites are core workflow steps.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [results] Robustness analysis varies fold count, sample subset, and hyper-parameter sharing to test stability of cross-validated performance.: [evidence span withheld — anchor status: not_found]
- [discussion] Low-metabolite-correlation metabolites lower the overall mean SCC; the model is not expected to predict every metabolite well.: "since not all metabolites may be associated with microbes, some metabolites will have lower prediction correlations, which resulted in an overall lower mean correlation across all metabolites."
