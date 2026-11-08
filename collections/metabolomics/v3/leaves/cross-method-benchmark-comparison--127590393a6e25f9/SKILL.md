---
name: cross-method-benchmark-comparison
description: Use when two or more microbiome-metabolome prediction models (e.g., MiMeNet
  vs. MelonnPan vs. NED) must be compared under an identical cross-validation protocol
  on paired community profiles.
when_to_use_negative:
- When only one method exists or the question is a single model's predictive accuracy
  rather than a head-to-head comparison — the cross-method setup adds no information.
- When datasets lack paired microbial and metabolite samples from the same specimens;
  cross-model benchmarking on mismatched pairs is meaningless.
- When the metabolites of interest are unannotated and cannot be evaluated by the
  annotated-metabolite-only protocols used in the MelonnPan comparison.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3697
tools:
- name: MelonnPan
  role: Baseline per-metabolite Elastic Net linear regression model whose predictions
    are benchmarked against MiMeNet under the same CV protocol.
  repo: https://github.com/biobakery/melonnpan
- name: MiMeNet
  role: Multi-task MLPNN predictor whose performance is evaluated and compared against
    MelonnPan and NED.
  repo: https://github.com/YDaiLab/MiMeNet
- name: BiomeNED
  role: Additional non-negative sparse NED encoder-decoder model included in supplementary
    cross-method comparison on the IBD External validation set.
  repo: https://github.com/vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:5e510215d18c5fac5ebd5a1501ba1c964486f1e5b311edca2a7b459a87b740da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/melonnpan
  tools:
  - BiomeNED
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cross-method-benchmark-comparison

## Summary

Head-to-head benchmarking of MiMeNet's MLPNN metabolite predictions against MelonnPan's Elastic Net linear models using shared 10-iteration 10-fold cross-validation across IBD (PRISM), cystic fibrosis, and soil datasets plus an external IBD test set.

## When to use

Use when two or more microbiome-metabolome prediction models (e.g., MiMeNet vs. MelonnPan vs. NED) must be compared under an identical cross-validation protocol on paired community profiles.

## When NOT to use

- When only one method exists or the question is a single model's predictive accuracy rather than a head-to-head comparison — the cross-method setup adds no information.
- When datasets lack paired microbial and metabolite samples from the same specimens; cross-model benchmarking on mismatched pairs is meaningless.
- When the metabolites of interest are unannotated and cannot be evaluated by the annotated-metabolite-only protocols used in the MelonnPan comparison.

## Inputs

- Paired microbiome-metabolome CSV matrices (samples x features) for IBD (PRISM), cystic fibrosis, and soil wetting datasets from the MiMeNet repository data directories
- External IBD validation dataset for held-out test predictions
- Identical 10 iterations of 10-fold CV sample partitions shared across methods
- Compositional feature normalization settings (RA or CLR) consistent with MelonnPan's relative-abundance requirement

## Outputs

- Per-metabolite Spearman correlation coefficients (SCC) for each method per dataset
- Well-predicted metabolite counts (SCC > 95th percentile of shuffled background) per method per dataset
- Aggregated mean SCC comparison table (e.g., melonnpan_scc_comparison.csv) with MiMeNet vs. linear-model baseline values
- Prediction correlation scatterplots comparing methods per dataset

## How to apply

Run both methods on the same 10 iterations of 10-fold cross-validation partitions derived from the paired microbiome-metabolome datasets (IBD PRISM, cystic fibrosis, soil wetting, and external IBD), normalizing compositional features consistently — MelonnPan requires relative abundance (RA) while MiMeNet accepts RA or centered log-ratio (CLR). For each metabolite, compute Spearman correlation coefficients (SCC) on held-out folds rather than in-sample fit, since SCC measures prediction accuracy. Establish a significance background by randomly shuffling samples in each cross-validated iteration, then call a metabolite 'well-predicted' when its SCC exceeds the 95th percentile of that shuffled background. Tune hyperparameters once from the first CV partition and share them across all iterations for both methods, so differences reflect model architecture (per-metabolite Elastic Net linear regression vs. multi-task MLPNN) rather than tuning advantage. Aggregate mean SCCs and well-predicted metabolite counts per dataset and compare against the reference values: MiMeNet raises mean SCC from 0.108 to 0.309 (IBD PRISM), 0.276 to 0.457 (CF), -0.272 to 0.264 (soil), and 0.168 to 0.275 (IBD External), and well-predicted counts from 198 to 366, 104 to 143, 4 to 29, and 186 to 308, respectively.

## Related tools

- **MelonnPan** (Baseline per-metabolite Elastic Net linear regression model whose predictions are benchmarked against MiMeNet under the same CV protocol.) — https://github.com/biobakery/melonnpan
- **MiMeNet** (Multi-task MLPNN predictor whose performance is evaluated and compared against MelonnPan and NED.) — https://github.com/YDaiLab/MiMeNet
- **BiomeNED** (Additional non-negative sparse NED encoder-decoder model included in supplementary cross-method comparison on the IBD External validation set.) — https://github.com/vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -output ./out -micro_norm RA -metab_norm RA -num_run_cv 10 -num_cv 10 -num_background 100 -threshold 0.95
```

## Evaluation signals

- Mean SCC matches reference values within reproducibility tolerance: 0.108 (MelonnPan) vs 0.309 (MiMeNet) for IBD PRISM; 0.276 vs 0.457 for cystic fibrosis; -0.272 vs 0.264 for soil; 0.168 vs 0.275 for IBD External.
- Well-predicted metabolite counts match: 198 vs 366 (IBD PRISM), 104 vs 143 (CF), 4 vs 29 (soil), 186 vs 308 (IBD External).
- SCC distributions are computed on held-out folds of the identical 10-iteration 10-fold CV partitions, not in-sample fit.
- Well-predicted metabolite threshold is the 95th percentile of a shuffled-sample background SCC distribution generated per cross-validated iteration.
- Hyperparameters are tuned once from the first CV partition and shared across all iterations and both methods.

## Limitations

- All methods including MiMeNet are still limited in providing biological plausibility and mechanistic insights.
- Not all metabolites may be associated with microbes, so some metabolites have lower prediction correlations, yielding an overall lower mean correlation across all metabolites.
- Higher threshold value observed for the soil data may be due to longitudinal observations (unresolved).
- Procedures to discern statistical association of microbe-metabolite interactions to host diseases do not yet exist.
- The reliance on annotated references can lead to poor performing models when annotations are missing or incorrect.

## Evidence

- [abstract] MiMeNet raises mean SCC from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264 on three paired datasets: "MiMeNet more accurately predicts metabolite abundances (mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264)"
- [abstract] Well-predicted metabolite counts increase from 198 to 366, 104 to 143, and 4 to 29: "identifies more well-predicted metabolites (increase in the number of well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29) compared to state-of-art linear models"
- [intro] MelonnPan uses Elastic Net linear regression to model relative abundance of each metabolite: "One method, MelonnPan, uses the Elastic Net linear regression to model the relative abundance of each metabolite using metagenomic taxonomic or functional features"
- [methods] Compositional transformation and low-abundance filtering precede cross-validated evaluation: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [methods] Well-predicted metabolite defined as SCC above 95th percentile: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [supplementary] Cross-validation uses 10 iterations of 10-fold with shared hyperparameters tuned from first partition: "Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition"
- [intro] MiMeNet package and datasets are freely available for reproduction: "The MiMeNet package and the datasets used for evaluation are freely available at https://github.com/YDaiLab/MiMeNet"
- [discussion] MiMeNet is distinct from MelonnPan's regularized linear regression: "The predictive model in MiMeNet distinguishes it from MelonnPan [26], which uses a regularized linear regression to model each metabolite separately."
- [readme] MelonnPan-Predict requires microbial sequence features relative abundances input: "MelonnPan-Predict workflow requires the following input:
    -   a table of microbial sequence features' relative abundances (samples in rows)"
