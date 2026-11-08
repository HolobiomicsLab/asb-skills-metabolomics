---
name: microbiome-metabolome-cross-validation
description: Use when paired microbial-feature and metabolite tables (samples-by-features
  CSV) must be evaluated by k-fold cross-validation and a principled significance
  cutoff for well-predicted metabolites is needed.
when_to_use_negative:
- The goal is linear-model (e.g., Elastic Net MelonnPan) baselines rather than MLPNN
  prediction — MelonnPan uses per-metabolite regularized regression with p-value/q-value
  significance, not shuffled SCC background cutoffs
- Only a single train/test split is available — the skill requires 10 iterations of
  10-fold cross-validation plus shuffled-label background iterations to build the
  empirical null
- Data are unpaired (microbial and metabolite samples do not correspond one-to-one),
  since cross-validation depends on paired samples-by-features matrices
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3172
tools:
- name: MiMeNet
  role: 'Primary framework: MLPNN trained under 10x10-fold cross-validation with shuffled-label
    background iterations and 95th-percentile SCC thresholding'
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Baseline linear (Elastic Net) comparator for well-predicted metabolite counts
  repo: https://github.com/biobakery/melonnpan
- name: BiomeNED
  role: Encoder-decoder neural network comparator for prediction correlation
  repo: https://github.com/vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:47a29fa4cb2c825f2d19f50c8ac6b391aa27af83b9ac6685bdbaa43bc06d5f13
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# microbiome-metabolome-cross-validation

## Summary

Cross-validated evaluation of microbiome-to-metabolome predictive models (e.g., MiMeNet MLPNN) using shuffled-label background distributions to define dataset-specific Spearman correlation cutoffs for well-predicted metabolites.

## When to use

Use when paired microbial-feature and metabolite tables (samples-by-features CSV) must be evaluated by k-fold cross-validation and a principled significance cutoff for well-predicted metabolites is needed.

## When NOT to use

- The goal is linear-model (e.g., Elastic Net MelonnPan) baselines rather than MLPNN prediction — MelonnPan uses per-metabolite regularized regression with p-value/q-value significance, not shuffled SCC background cutoffs
- Only a single train/test split is available — the skill requires 10 iterations of 10-fold cross-validation plus shuffled-label background iterations to build the empirical null
- Data are unpaired (microbial and metabolite samples do not correspond one-to-one), since cross-validation depends on paired samples-by-features matrices

## Inputs

- Comma-delimited samples-by-microbial-features matrix (e.g., taxonomic abundances) for IBD (PRISM), cystic fibrosis, or soil datasets
- Comma-delimited samples-by-metabolomic-features matrix
- JSON file of network hyperparameters (-net_params)
- Optional external validation microbial/metabolite matrices (-external_micro, -external_metab)

## Outputs

- Per-metabolite Spearman correlation coefficient (SCC) evaluation values from cross-validated and shuffled-background iterations
- Dataset-specific 95th-percentile SCC significance cutoff (e.g., 0.136 IBD, 0.129 CF, 0.410 soil)
- List/counts of well-predicted metabolites per dataset (e.g., 366, 143, 29)

## How to apply

Load paired samples-by-microbe and samples-by-metabolite CSV matrices and transform them with MiMeNet's compositional normalization (relative abundance or center log-ratio), optionally filtering low-abundance features. Run 10 iterations of 10-fold cross-validation (num_run_cv/num_cv, num_background for shuffled iterations), training the MLPNN with a JSON hyperparameters file; separately run the background procedure where sample labels are randomly shuffled each iteration to build an empirical null distribution of per-metabolite Spearman correlation coefficients (SCC). Compute the 95th percentile of each dataset's shuffled background SCC distribution as the significance cutoff, then mark unshuffled metabolites whose SCC exceeds that cutoff as well-predicted. Dataset-specific 95th-percentile cutoffs reported were 0.136 (IBD/PRISM), 0.129 (cystic fibrosis), and 0.410 (soil), yielding 366, 143, and 29 well-predicted metabolites for MiMeNet versus 198, 104, and 4 for linear models. Judge correctness by whether derived cutoffs and well-predicted counts reproduce the published values within the stochastic variation of the shuffled background runs.

## Related tools

- **MiMeNet** (Primary framework: MLPNN trained under 10x10-fold cross-validation with shuffled-label background iterations and 95th-percentile SCC thresholding) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Baseline linear (Elastic Net) comparator for well-predicted metabolite counts) — https://github.com/biobakery/melonnpan
- **BiomeNED** (Encoder-decoder neural network comparator for prediction correlation) — https://github.com/vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -output out/ -net_params net_params.json -micro_norm RA -metab_norm RA -num_background 10 -num_run_cv 10 -num_cv 10
```

## Evaluation signals

- The 95th-percentile cutoff of the shuffled background SCC distribution should approximate the reported dataset-specific values (0.136 IBD/PRISM, 0.129 cystic fibrosis, 0.410 soil) within stochastic variation
- Well-predicted metabolite counts should approach 366 (IBD), 143 (CF), and 29 (soil) for MiMeNet, versus 198, 104, 4 for linear models
- Mean cross-validated SCC should match reported values (0.309, 0.457, 0.264 for MiMeNet on the three datasets)
- Shuffled-label background runs must produce SCC distributions centered near zero; the unshuffled evaluations must be compared against the same background per dataset
- Normalization choice (RA vs CLR) should be fixed via -micro_norm/-metab_norm and applied identically across CV and background iterations

## Limitations

- The soil dataset's higher cutoff may stem from longitudinal observations (unresolved in the paper)
- Not all metabolites may be associated with microbes, so some metabolites have low prediction correlations, lowering overall mean SCC
- Results depend on the number of shuffled background iterations (num_background); with too few iterations the 95th-percentile estimate is noisy

## Evidence

- [methods] Cross-validated evaluation and shuffled background define SCC cutoff: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [methods] 95th-percentile threshold rule: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [abstract] CV scheme on three datasets: "Using ten iterations of 10-fold cross-validation on three paired microbiome-metabolome datasets"
- [methods] Compositional transformation and filtering: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [abstract] Well-predicted counts vs linear models: "identifies more well-predicted metabolites (increase in the number of well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29)"
- [readme] README CLI parameters: "-num_background NUM_BACKGROUND    Number of background CV Iterations"
