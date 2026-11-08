---
name: bootstrap-confidence-interval-estimation
description: Use when reporting top-1/top-10 candidate ranking accuracies from a spectral
  retrieval benchmark and you need statistically grounded 95% confidence intervals
  rather than point estimates.
when_to_use_negative:
- If you only need the raw point-estimate top-k accuracies without any uncertainty
  quantification (e.g., for a quick sanity check).
- If your task is spectrum prediction error (e.g., cosine similarity of predicted
  vs. observed spectra) rather than candidate retrieval ranking, where a different
  metric-family bootstrap would be needed.
- If the test set is so small that bootstrap resampling of hit/miss indicators yields
  degenerate (all-hit or all-miss) samples, making intervals meaningless.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ICEBERG
  role: Source model whose retrieval top-k accuracies are being interval-estimated
  repo: https://github.com/samgoldman97/ms-pred
- name: SCARF
  role: Baseline model whose retrieval top-k accuracies are being interval-estimated
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:01d30d4d854727b4399efbb5cfb264e12d92349d899d993486d9583191b7450e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MetFrag
  - Python
  - SCARF
  - uv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bootstrap-confidence-interval-estimation

## Summary

Estimate bootstrap 95% confidence intervals for retrieval top-k accuracy metrics (top-1/top-10 hit rates) when benchmarking ICEBERG against baseline spectrum prediction models on NPLIB1, NIST20, and CASMI22 candidate ranking tasks.

## When to use

Use when reporting top-1/top-10 candidate ranking accuracies from a spectral retrieval benchmark and you need statistically grounded 95% confidence intervals rather than point estimates.

## When NOT to use

- If you only need the raw point-estimate top-k accuracies without any uncertainty quantification (e.g., for a quick sanity check).
- If your task is spectrum prediction error (e.g., cosine similarity of predicted vs. observed spectra) rather than candidate retrieval ranking, where a different metric-family bootstrap would be needed.
- If the test set is so small that bootstrap resampling of hit/miss indicators yields degenerate (all-hit or all-miss) samples, making intervals meaningless.

## Inputs

- Per-spectrum retrieval ranking results (candidate hit/miss indicators per model, per dataset)
- Top-1/top-10 hit-rate outputs from retrieval_benchmark_torchmetrics.py
- retrieval_accuracy_table.csv rows to annotate with intervals

## Outputs

- Bootstrap 95% confidence intervals for top-k accuracy per model and dataset
- Annotated retrieval_accuracy_table.csv with interval bounds

## How to apply

After ranking candidate structures per test spectrum by spectral similarity (e.g., via run_scripts/iceberg/06_run_retrieval.py), compute top-1/top-10 hit rates with retrieval_benchmark_torchmetrics.py, then run src/ms_pred/retrieval/bootstrap_metrics.py to resample the per-spectrum hit indicators and obtain 95% bootstrap confidence intervals around each top-k accuracy. This lets you quantify whether observed margins, such as ICEBERG's ~29% vs 20% top-1 on NPLIB1 or 12.9% vs 8.6% top-1 on CASMI22, are separable given the test set size. Apply the same procedure per model and per dataset (NPLIB1, NIST20, CASMI22) so confidence intervals accompany every cell of the retrieval_accuracy_table.csv.

## Related tools

- **ICEBERG** (Source model whose retrieval top-k accuracies are being interval-estimated) — https://github.com/samgoldman97/ms-pred
- **SCARF** (Baseline model whose retrieval top-k accuracies are being interval-estimated) — https://github.com/samgoldman97/ms-pred

## Examples

```
python src/ms_pred/retrieval/bootstrap_metrics.py --results retrieval_accuracy_table.csv
```

## Evaluation signals

- Bootstrap lower and upper 95% bounds should bracket the point-estimate top-k accuracy for every model/dataset cell.
- Interval width should shrink (narrower bounds) as the number of test spectra increases.
- All models/datasets in retrieval_accuracy_table.csv should carry interval bounds, not just the headline ICEBERG result.
- Re-running bootstrap_metrics.py with a different random seed should shift bounds by at most a few tenths of a percent, confirming convergence.

## Limitations

- Bootstrap resampling assumes the test spectra are exchangeable; correlated spectra (e.g., similar structures) may yield overly narrow intervals.
- The skill is scoped to top-k retrieval accuracy metrics; applying it to other metric families requires adapting the resampling unit.
- Confidence intervals do not correct for candidate-library construction biases (e.g., the 50-candidate true-plus-49-decoy setup).

## Evidence

- [methods] Bootstrap 95% CIs accompany top-k accuracies: [evidence span withheld — anchor status: not_found]
- [readme] Confidence intervals ground the retrieval benchmarking workflow: "running `src/ms_pred/retrieval/bootstrap_metrics.py`"
- [results] ICEBERG retrieval margins reported relative to baselines: [evidence span withheld — anchor status: not_found]
