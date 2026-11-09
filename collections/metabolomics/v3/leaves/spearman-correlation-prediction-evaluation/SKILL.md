---
name: spearman-correlation-prediction-evaluation
description: Use when reproducing or benchmarking metabolite abundance prediction
  from paired microbiome-metabolome count data and per-metabolite SCC values, mean
  SCC, and well-predicted metabolite counts are needed.
when_to_use_negative:
- When the goal is conditional probability or per-feature association discovery (e.g.,
  mmvec-style microbe-metabolite co-occurrence) rather than predicting the entire
  metabolomic profile from microbial features
- When the data are single-omics (microbiome-only or metabolome-only) with no paired
  samples, since cross-validated SCC evaluation requires metabolite ground truth per
  sample
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MiMeNet
  role: MLPNN used to predict the community metabolome from microbial taxonomic features
    and produce cross-validated per-metabolite SCC evaluations
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Baseline regularized linear regression predictor used for comparison of metabolite
    prediction correlations
  repo: https://github.com/biobakery/melonnpan
- name: BiomeNED
  role: Neural encoder-decoder baseline whose metabolite prediction correlations are
    compared to MiMeNet's on the IBD (External) dataset
  repo: https://github.com/vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:ab2f1e692dc1cf984a28b7df3b4213f861f0def2ce73a7545fe35bd8fb0cf04f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - BiomeNED
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spearman-correlation-prediction-evaluation

## Summary

Evaluates microbiome-to-metabolome prediction models (e.g., MiMeNet) using per-metabolite Spearman correlation coefficients (SCC) under 10 iterations of 10-fold cross-validation, and flags well-predicted metabolites against a shuffled-sample background.

## When to use

Use when reproducing or benchmarking metabolite abundance prediction from paired microbiome-metabolome count data and per-metabolite SCC values, mean SCC, and well-predicted metabolite counts are needed.

## When NOT to use

- When the goal is conditional probability or per-feature association discovery (e.g., mmvec-style microbe-metabolite co-occurrence) rather than predicting the entire metabolomic profile from microbial features
- When the data are single-omics (microbiome-only or metabolome-only) with no paired samples, since cross-validated SCC evaluation requires metabolite ground truth per sample

## Inputs

- Comma-delimited CSV of samples by microbial count features (e.g., data/IBD/microbiome_PRISM.csv)
- Comma-delimited CSV of samples by metabolomic count features (e.g., data/IBD/metabolome_PRISM.csv)
- External test set microbial and metabolomic CSVs (e.g., microbiome_external.csv, metabolome_external.csv)
- Metabolite annotation CSV for the annotated-metabolite subset
- JSON file of neural network hyper-parameters (number of layers, layer size, L2 penalty, dropout rate)

## Outputs

- Per-metabolite Spearman correlation coefficients (SCC) from cross-validated evaluation
- Shuffled-sample background SCC distribution per metabolite
- Set of well-predicted metabolites with SCC above the 95th percentile of the background
- Per-dataset summary table of mean SCC and well-predicted metabolite counts (e.g., mimenet_scc_summary.csv)

## How to apply

Run MiMeNet_train.py on paired microbiome/metabolome CSVs (e.g., IBD PRISM, cystic fibrosis, soil) with num_run_cv set to 10 iterations and num_cv to 10 folds, so models are evaluated cross-validated, yielding a SCC for each metabolite. Before training, apply MiMeNet's compositional transformation (relative abundance or CLR, or 'None' if data are already transformed) and filter low-abundant microbial and metabolite features; reuse tuned network hyper-parameters (number of layers, layer size, L2 penalty, dropout rate) stored in network_parameters.txt across iterations. For the IBD dataset, pass the external IBD test set via -external_micro and -external_metab to obtain external-validation SCCs. To identify well-predicted metabolites, generate an empirical background by running additional cross-validated iterations on randomly shuffled samples (num_background, at least 10) and take metabolites whose SCC exceeds the 95th percentile of the background distribution. Report mean SCC per dataset (restricted to annotated metabolites for IBD) and count well-predicted metabolites; compare against published values (mean SCC 0.309 IBD annotated, 0.457 CF, 0.264 soil, 0.275 IBD External; well-predicted counts 366, 143, 29).

## Related tools

- **MiMeNet** (MLPNN used to predict the community metabolome from microbial taxonomic features and produce cross-validated per-metabolite SCC evaluations) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Baseline regularized linear regression predictor used for comparison of metabolite prediction correlations) — https://github.com/biobakery/melonnpan
- **BiomeNED** (Neural encoder-decoder baseline whose metabolite prediction correlations are compared to MiMeNet's on the IBD (External) dataset) — https://github.com/vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -external_micro data/IBD/microbiome_external.csv -external_metab data/IBD/metabolome_external.csv -micro_norm None -metab_norm CLR -net_params results/IBD/network_parameters.txt -annotation data/IBD/metabolome_annotation.csv -num_run_cv 10 -output IBD
```

## Evaluation signals

- Reproduced mean SCC falls within tolerance of published values: 0.309 (IBD annotated), 0.457 (cystic fibrosis), 0.264 (soil), 0.275 (IBD External)
- Well-predicted metabolite counts (SCC above 95th percentile of shuffled background) are of the same order as reported: 366 (IBD), 143 (CF), 29 (soil)
- The per-dataset summary table (e.g., mimenet_scc_summary.csv) exists with mean SCC and well-predicted counts for each of the three datasets
- Background SCC generation used at least 10 iterations of 10-fold cross-validation on shuffled samples
- Shared hyper-parameters tuned from the first partition were reused across the 10 CV iterations

## Limitations

- Not all metabolites may be associated with microbes, so some metabolites have lower prediction correlations, resulting in a lower overall mean correlation across all metabolites
- A higher threshold value was observed for the soil data, possibly due to longitudinal observations (unresolved)
- Prior annotation-based approaches can perform poorly when annotations are missing or incorrect; MiMeNet's IBD evaluation on annotated metabolites benefited from also predicting unannotated metabolites

## Evidence

- [abstract] MiMeNet more accurately predicts metabolite abundances (mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264): "mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264"
- [methods] Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [readme] MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted.: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted"
- [readme] Number of iterations for cross-validation parameter description from README usage: "num_run_cv | Parameter to specify how many iterations of cross-validated evaluation to perform"
- [abstract] increase in the number of well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29: "identifies more well-predicted metabolites (increase in the number of well-predicted metabolites from 198 to 366, 104 to 143, and 4 to 29)"
