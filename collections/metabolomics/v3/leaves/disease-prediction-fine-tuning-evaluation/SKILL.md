---
name: disease-prediction-fine-tuning-evaluation
description: Use when fine-tuning a pre-trained metabolomic language model on binary
  chronic-disease endpoints from UK Biobank NMR metabolomics and benchmarking AUC
  against ML baselines with paired significance tests.
when_to_use_negative:
- The metabolite feature table still contains missing values and has not been imputed
  — run the 'metabolome-transformer impute' subcommand first, since the input table
  must contain the metabolite columns listed in metabolite_columns.txt
- The task is risk stratification / MetaboRS score derivation from already fine-tuned
  prediction probabilities rather than fine-tuning and AUC benchmarking itself
- The goal is architectural ablation (e.g. removing the metabolite correlation matrix
  from attention during pre-training) rather than downstream fine-tuning and baseline
  comparison
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2269
tools:
- name: MetaboLM
  role: Pre-trained metabolomic language model that is fine-tuned per disease endpoint
    to produce binary disease-prediction classifiers
  repo: github.com/Qiu-Shizheng/MetaboLM
- name: Metabolome-Transformer
  role: CLI toolkit (metabolome-transformer train-binary / predict-binary) for fine-tuning
    on binary disease labels and generating probability predictions from metabolite
    tables
  repo: github.com/Qiu-Shizheng/Metabolome-Transformer
- name: PyTorch
  role: Deep-learning backend used to fine-tune MetaboLM and train the MLP and FT-Transformer
    baselines
- name: CatBoost
  role: Gradient-boosting baseline classifier trained on the same 16 disease endpoints
    and train/test splits
- name: scikit-learn
  role: Baseline MLP implementation and AUC computation on test splits
- name: SciPy
  role: Paired t-tests across the 16 disease endpoints comparing MetaboLM AUC against
    each baseline
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:4350cfa9747e141f085f0bbc3340160677d78f0b3657ff4a19dfeb8610f85f20
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  tools:
  - CatBoost
  - MetaboLM
  - MetaboLM (github.com/Qiu-Shizheng/MetaboLM)
  - Metabolome-Transformer
  - PyTorch
  - SciPy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# disease-prediction-fine-tuning-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tune the MetaboLM metabolomic language model (pre-trained on NMR plasma metabolomics from 83,744 relatively healthy UK Biobank participants) on individuals diagnosed with 16 common chronic diseases, then benchmark its test-set AUC against tuned MLP, CatBoost and FT-Transformer baselines using paired t-tests across the 16 endpoints.

## When to use

Use when fine-tuning a pre-trained metabolomic language model on binary chronic-disease endpoints from UK Biobank NMR metabolomics and benchmarking AUC against ML baselines with paired significance tests.

## When NOT to use

- The metabolite feature table still contains missing values and has not been imputed — run the 'metabolome-transformer impute' subcommand first, since the input table must contain the metabolite columns listed in metabolite_columns.txt
- The task is risk stratification / MetaboRS score derivation from already fine-tuned prediction probabilities rather than fine-tuning and AUC benchmarking itself
- The goal is architectural ablation (e.g. removing the metabolite correlation matrix from attention during pre-training) rather than downstream fine-tuning and baseline comparison

## Inputs

- Pre-trained MetaboLM checkpoint (pre-trained on NMR plasma metabolomics of 83,744 relatively healthy UK Biobank participants)
- Fine-tuning cohorts of individuals diagnosed with each of the 16 common chronic diseases during UK Biobank follow-up
- CSV training tables containing sample ID column, metabolite columns (249 metabolites per assets/pretrained/metabolite_columns.txt), and a binary label column
- Imputed metabolite feature tables (missing values imputed beforehand via metabolome-transformer impute)

## Outputs

- Per-disease fine-tuned classifiers (e.g. results/current_task/classifier.pt)
- Prediction CSV with ID column, probability, and binary prediction
- benchmark_comparison.csv with per-disease AUC for MetaboLM vs MLP, CatBoost, FT-Transformer and paired t-test statistics

## How to apply

Start from the MetaboLM checkpoint pre-trained on the 83,744-participant NMR metabolomics cohort, and fine-tune it separately on each of the 16 common chronic disease endpoints identified during UK Biobank follow-up. Prepare each training file with a sample ID column, the metabolite columns, and a binary label column, then run the 'metabolome-transformer train-binary' command (e.g. --task current or --task future, --epochs 10, --batch-size 12, with --device cpu if no GPU). Train MLP, CatBoost and FT-Transformer baselines on the same 16 endpoints under identical train/test splits, compute test-set AUC per endpoint for every model, and run paired t-tests across the 16 endpoints comparing MetaboLM to each baseline. Expected outcome from the source study: MetaboLM beats MLP in 16/16 diseases (t = 6.61, P = 8.22E-06), FT-Transformer in 14/16 (t = 3.95, P = 0.00127), and CatBoost in only 9/16 (t = -0.93, P = 0.366, not significant). Write the per-disease AUC table and t-test statistics to benchmark_comparison.csv.

## Related tools

- **MetaboLM** (Pre-trained metabolomic language model that is fine-tuned per disease endpoint to produce binary disease-prediction classifiers) — github.com/Qiu-Shizheng/MetaboLM
- **Metabolome-Transformer** (CLI toolkit (metabolome-transformer train-binary / predict-binary) for fine-tuning on binary disease labels and generating probability predictions from metabolite tables) — github.com/Qiu-Shizheng/Metabolome-Transformer
- **PyTorch** (Deep-learning backend used to fine-tune MetaboLM and train the MLP and FT-Transformer baselines)
- **CatBoost** (Gradient-boosting baseline classifier trained on the same 16 disease endpoints and train/test splits)
- **scikit-learn** (Baseline MLP implementation and AUC computation on test splits)
- **SciPy** (Paired t-tests across the 16 disease endpoints comparing MetaboLM AUC against each baseline)

## Examples

```
metabolome-transformer train-binary --input train.csv --output-dir results/current_task --id-column sample_id --label-column label --task current --epochs 10 --batch-size 12
```

## Evaluation signals

- benchmark_comparison.csv contains per-disease test-set AUC for all 4 models across 16 endpoints plus the three paired t-test statistics
- T-test results reproduce the reported pattern: MetaboLM > MLP in 16/16 diseases (t = 6.61, P = 8.22E-06), > FT-Transformer in 14/16 (t = 3.95, P = 0.00127), > CatBoost in only 9/16 (t = -0.93, P = 0.366)
- Prediction output CSV has the required schema: ID column, probability, and prediction
- Baselines are trained on identical train/test splits as MetaboLM so the paired t-test across endpoints is valid
- Fine-tuned classifier artifacts (classifier.pt) are written to the specified --output-dir for each task

## Limitations

- MetaboLM did not significantly outperform CatBoost (t = -0.93, P = 0.366), so superiority claims are only supported against MLP and FT-Transformer
- The UK Biobank cohort is predominantly of European descent, healthier, and of higher socioeconomic status, which may limit generalizability of the fine-tuned disease predictors
- NMR-based metabolomics may overlook certain metabolites detectable by other analytical methods, bounding the feature space to the 249 metabolite columns

## Evidence

- [abstract] Fine-tuning with metabolomics data from individuals diagnosed with 16 common chronic diseases: [evidence span withheld — anchor status: not_found]
- [intro] we fine-tuned the pre-trained model using samples from individuals diagnosed with 16 common chronic diseases: "we fine-tuned the pre-trained model using samples from individuals diagnosed with 16 common chronic diseases during the follow-up period"
- [readme] The training file should contain the sample ID column, metabolite columns, and a binary label column.: [evidence span withheld — anchor status: external_doc]
- [readme] Prediction output contains the ID column, probability, and prediction.: [evidence span withheld — anchor status: external_doc]
