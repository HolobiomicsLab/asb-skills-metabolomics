---
name: fragmentation-depth-ablation-analysis
description: Use when evaluating how a fragmentation depth parameter (e.g. d=3 vs
  d=4) changes MS/MS spectrum prediction metrics, or when reproducing reported ablation
  deltas across metric splits.
when_to_use_negative:
- Input is a single model's spectrum prediction output rather than a published ablation
  pair (D3 vs D4) — use spectrum-level evaluation instead
- The ablation involves a parameter other than fragmentation depth (e.g. latent entropy
  regularization or hydrogen tolerance j) — build a separate delta table rather than
  reusing this skill's fixed D4 vs D3 extraction
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Compute deltas between D4 and D3 Table 1 values and validate against published
    figures
- name: Pandas
  role: Assemble extracted metrics, standard deviations, and deltas into a structured
    CSV delta table
- name: NumPy
  role: Numeric delta computation and validation of positive-delta invariant across
    all metric/split combinations
- name: FraGNNet
  role: Source of the D3/D4 ablation being compared; models were run on NIST20 MS/MS
    with d in {3,4} and j=4
  repo: https://github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:489611d0af33cf9545a113708e143d72711d86545c2388bcf13d91d0d2411ea3
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/FraGNNet/fragnnet
  tools:
  - FraGNNet
  - NumPy
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fragmentation-depth-ablation-analysis

## Summary

Extracts and compares prediction performance of FraGNNet-D4 vs FraGNNet-D3 from published Table 1 figures to quantify how fragmentation depth (d=3 vs d=4) affects C2MS spectrum prediction on NIST20 MS/MS.

## When to use

Use when evaluating how a fragmentation depth parameter (e.g. d=3 vs d=4) changes MS/MS spectrum prediction metrics, or when reproducing reported ablation deltas across metric splits.

## When NOT to use

- Input is a single model's spectrum prediction output rather than a published ablation pair (D3 vs D4) — use spectrum-level evaluation instead
- The ablation involves a parameter other than fragmentation depth (e.g. latent entropy regularization or hydrogen tolerance j) — build a separate delta table rather than reusing this skill's fixed D4 vs D3 extraction

## Inputs

- Published Table 1 rows for FraGNNet-D4 and FraGNNet-D3 on NIST20 MS/MS (mean and std over 5 random seeds)
- Metrics: C_BIN and C_HUN on InChIKey and Scaffold splits

## Outputs

- CSV delta table: per metric/split values, standard deviations, and D4−D3 deltas (+0.015 InChIKey C_BIN, +0.016 InChIKey C_HUN, +0.019 Scaffold C_BIN, +0.021 Scaffold C_HUN)

## How to apply

Load published Table 1 values for FraGNNet-D4 and FraGNNet-D3 on the NIST20 MS/MS dataset; these are mean±std over 5 random seeds for C_BIN and C_HUN on InChIKey and Scaffold splits. Extract the four values per model: FraGNNet-D4 (0.736±0.002, 0.709±0.002 InChIKey; 0.678±0.005, 0.654±0.006 Scaffold) and FraGNNet-D3 (0.721±0.002, 0.693±0.002 InChIKey; 0.659±0.001, 0.633±0.001 Scaffold). Compute deltas (D4−D3) per metric/split to confirm the +0.015 to +0.021 improvement from increasing d from 3 to 4. Validate that deltas exceed zero for all four metric-split combinations and match published figures exactly.

## Related tools

- **Python** (Compute deltas between D4 and D3 Table 1 values and validate against published figures)
- **Pandas** (Assemble extracted metrics, standard deviations, and deltas into a structured CSV delta table)
- **NumPy** (Numeric delta computation and validation of positive-delta invariant across all metric/split combinations)
- **FraGNNet** (Source of the D3/D4 ablation being compared; models were run on NIST20 MS/MS with d in {3,4} and j=4) — https://github.com/FraGNNet/fragnnet

## Examples

```
pandas.read_csv('table1.csv') ; deltas = df[df.model=='FraGNNet-D4'].metric_vals.values - df[df.model=='FraGNNet-D3'].metric_vals.values ; assert (deltas > 0).all()
```

## Evaluation signals

- Deltas must be positive for all four metric/split combinations (C_BIN InChIKey, C_HUN InChIKey, C_BIN Scaffold, C_HUN Scaffold)
- D4 InChIKey C_BIN must equal 0.736±0.002 and D3 must equal 0.721±0.002 — the extracted values must match published Table 1 exactly
- Scaffold deltas should exceed InChIKey deltas for both metrics (larger effect on the harder split)
- All reported values must be mean±std over exactly 5 random seeds; no seed should be silently dropped
- Metric ordering must be consistent: C_BIN > C_HUN for both models on both splits

## Limitations

- The ablation only covers fragmentation depth d ∈ {3,4}; other depth values or hydrogen tolerances j ≠ 4 are not reported in this table
- Reported values are specific to the NIST20 MS/MS dataset with the merged-spectrum preprocessing; generalization to other instruments/adducts is unverified in this ablation
- Standard deviations are small (≤0.006), so deltas of ~0.015–0.021 exceed seed noise, but no formal significance test is provided in Table 1

## Evidence

- [methods] Fragmentation depth defined as ablation parameter: [evidence span withheld — anchor status: not_found]
- [methods] Table 1 metrics used for comparison: [evidence span withheld — anchor status: not_found]
- [methods] Repository for reproducing FraGNNet-D3/D4 runs: [evidence span withheld — anchor status: not_found]
