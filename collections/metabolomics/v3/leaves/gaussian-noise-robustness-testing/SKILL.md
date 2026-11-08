---
name: gaussian-noise-robustness-testing
description: Use when a paired microbiome-metabolome model has been trained in two
  configurations (full vs. annotated-subset outputs) and you must quantify whether
  multivariate learning confers robustness to input perturbation before comparing
  per-metabolite SCCs.
when_to_use_negative:
- When the input metabolome contains no unannotated metabolites — there is no full-vs-annotated
  contrast to test, so multivariate-transfer benefit cannot be assessed.
- When the analysis goal is direct method benchmarking against MelonnPan or NED rather
  than testing the noise robustness of the multivariate-learning hypothesis.
- When inputs are already perturbed or quality-filtered metagenomic features without
  the original abundance matrix — additive Gaussian noise re-perturbation on transformed
  data would confound the CLR/None preprocessing.
tools:
- name: MiMeNet
  role: Trains the full-metabolite and annotated-only MLPNN models with matched cross-validation
    settings, and provides the perturb-and-re-evaluate framework for the Gaussian-noise
    robustness comparison.
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:c0c3526fa6b9644de31ca5a4ea702c3ec6e5dd6c690e438476c4dcabcc11f151
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Gaussian-noise robustness testing

## Summary

A robustness evaluation protocol for microbiome→metabolome predictors (MiMeNet) in which trained models are re-evaluated on microbiome input profiles perturbed with additive Gaussian noise, to test whether multi-task learning on the full metabolite set (8848 metabolites) yields a model more robust than one trained only on the 466 annotated metabolites.

## When to use

Use when a paired microbiome-metabolome model has been trained in two configurations (full vs. annotated-subset outputs) and you must quantify whether multivariate learning confers robustness to input perturbation before comparing per-metabolite SCCs.

## When NOT to use

- When the input metabolome contains no unannotated metabolites — there is no full-vs-annotated contrast to test, so multivariate-transfer benefit cannot be assessed.
- When the analysis goal is direct method benchmarking against MelonnPan or NED rather than testing the noise robustness of the multivariate-learning hypothesis.
- When inputs are already perturbed or quality-filtered metagenomic features without the original abundance matrix — additive Gaussian noise re-perturbation on transformed data would confound the CLR/None preprocessing.

## Inputs

- data/IBD/microbiome_PRISM.csv — samples-by-microbial-features matrix (comma delimited)
- data/IBD/metabolome_PRISM.csv — samples-by-metabolomic-features matrix (comma delimited)
- data/IBD/metabolome_annotation.csv — annotation file defining the 466 annotated metabolite subset
- results/IBD/network_parameters.txt — JSON file of tuned network hyperparameters
- Trained MiMeNet models (full-metabolite and annotated-only configurations)

## Outputs

- annotated_vs_full_scc_comparison.csv — per-metabolite SCC comparison table plus well-predicted metabolite counts
- Per-metabolite Spearman correlation coefficients (SCC) for the annotated subset under Gaussian-noise perturbation
- Wilcoxon signed-rank P-value comparing paired SCCs between the two models

## How to apply

Train two MiMeNet MLPNN models with identical settings: one on the full 8848-metabolite output layer and one restricted to the 466 annotated metabolites listed in data/IBD/metabolome_annotation.csv, both using the tuned hyperparameters in results/IBD/network_parameters.txt and 10 iterations of 10-fold cross-validation (num_run_cv=10, num_cv=10). Preprocess identically per MiMeNet preprocessing: 'None' normalization for the microbiome (data/IBD/microbiome_PRISM.csv), CLR transformation for the metabolome (data/IBD/metabolome_PRISM.csv), and filtering of low-abundance microbial and metabolite features. Then perturb the input microbial profiles with additive Gaussian noise and re-evaluate both models on the perturbed inputs. Compute per-metabolite Spearman correlation coefficients (SCC) on the annotated 466-metabolite subset for both models, count well-predicted metabolites as those whose SCC exceeds the 95th percentile of the shuffled-sample empirical background (num_background >= 10), and test paired SCC differences between the full-metabolome and annotated-only models with a Wilcoxon signed-rank test, writing annotated_vs_full_scc_comparison.csv.

## Related tools

- **MiMeNet** (Trains the full-metabolite and annotated-only MLPNN models with matched cross-validation settings, and provides the perturb-and-re-evaluate framework for the Gaussian-noise robustness comparison.) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -annotation data/IBD/metabolome_annotation.csv -net_params results/IBD/network_parameters.txt -micro_norm None -metab_norm CLR -num_run_cv 10 -num_cv 10 -num_background 10 -output results/IBD_full/
```

## Evaluation signals

- Annotated-subset mean SCC must increase from 0.259 (annotated-only model) to 0.309 (full-metabolite model).
- Wilcoxon signed-rank test on paired annotated-subset SCCs must yield P < 10^-47.
- Number of well-predicted annotated metabolites (SCC above the 95th percentile of the shuffled-sample background) must increase from 333 to 366.
- Both models must be trained with identical cross-validation settings (num_run_cv=10, num_cv=10) and the same hyperparameters from results/IBD/network_parameters.txt so the contrast isolates the multivariate-learning effect.
- Well-predicted count uses num_background >= 10 shuffled iterations to establish the empirical 95th-percentile threshold.

## Limitations

- Additive Gaussian noise is a synthetic perturbation; robustness under real measurement variation is not directly measured by this protocol.
- The comparison is restricted to the 466 annotated metabolites; the full model's behavior on the remaining unannotated metabolites is not validated by this test.
- Not all metabolites may be microbe-associated, so some metabolites will have lower prediction correlations regardless of model configuration.

## Evidence

- [methods] Gaussian noise protocol: [evidence span withheld — anchor status: not_found]
- [discussion] Multivariate learning benefit: "the MiMeNet prediction on the set of the annotated metabolites benefited from including tasks of predicting the rest of the unannotated metabolites in the metabolome profiles (Fig 2D–2F)."
- [methods] Preprocessing and filtering: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [methods] SCC cross-validated evaluation: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [readme] Normalization options: "-micro_norm MICRO_NORM            Microbiome normalization (RA, CLR, or None)"
- [methods] Well-predicted threshold: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
