---
name: oracle-vs-predictive-retrieval-gap-analysis
description: Use when a two-stage retrieval-then-generation system has both oracle
  and predicted retrieval results on the same test set and you need to decompose end-to-end
  error into retrieval-error vs generation-error contributions.
when_to_use_negative:
- Only one retrieval condition is available (oracle or predictive alone) — the gap
  cannot be computed and no error decomposition is possible
- The analysis goal is the generator's absolute accuracy rather than the retrieval-error
  penalty — report the per-condition metrics directly instead of differencing them
- The comparison method reports no similarity or MCES measures (e.g. MSNovelist results
  are accuracy-only, denoted '-') — restrict the gap table to the metrics actually
  available per method
tools:
- name: Python
  role: Computes the per-dataset oracle-minus-predictive gaps for each metric and
    assembles the joined gap-vs-SPA CSV table
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) for Tanimoto similarity
    used as one of the gap metrics
  repo: https://github.com/HassounLab/MADGEN
- name: MADGEN
  role: Produces the oracle-retriever and predictive-retriever generation outputs
    being compared; oracle scaffold comes from the lookup table
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:3223f6c7441294baf9d6e8aa0fffd1e361c1201813788d2fe218b514df21a05b
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - Python
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Oracle-vs-predictive retrieval gap analysis

## Summary

Quantifies how much MADGEN's de novo generation accuracy drops when a predicted scaffold (contrastive-ranking retriever) replaces the oracle scaffold lookup, and relates the gap to the retriever's scaffold prediction accuracy (SPA) across NIST23, CANOPUS, and MassSpecGym.

## When to use

Use when a two-stage retrieval-then-generation system has both oracle and predicted retrieval results on the same test set and you need to decompose end-to-end error into retrieval-error vs generation-error contributions.

## When NOT to use

- Only one retrieval condition is available (oracle or predictive alone) — the gap cannot be computed and no error decomposition is possible
- The analysis goal is the generator's absolute accuracy rather than the retrieval-error penalty — report the per-condition metrics directly instead of differencing them
- The comparison method reports no similarity or MCES measures (e.g. MSNovelist results are accuracy-only, denoted '-') — restrict the gap table to the metrics actually available per method

## Inputs

- Reproduced per-condition metric tables (oracle-retriever and predictive-retriever runs) for NIST23, CANOPUS, and MassSpecGym with Top-1, Top-10, Tanimoto, and MCES columns
- Scaffold prediction accuracy (SPA) values for the predictive retriever per dataset
- Generated-molecule CSVs with columns true, pred, scaffold as expected by evaluation_generation.py

## Outputs

- CSV table with columns dataset, metric, oracle value, predictive value, gap, SPA
- Qualitative interpretation of the gap-vs-SPA relationship across the three datasets

## How to apply

Load the reproduced per-condition metric tables for the oracle-retriever run (lookup-table scaffold) and the predictive-retriever run (contrastive scaffold ranking) on each dataset. In Python, compute the oracle-minus-predictive gap per dataset for each metric: Top-1 accuracy, Top-10 accuracy, Tanimoto similarity from Morgan fingerprints (radius 2, 2048 bits, via RDKit), and MCES edit distance. Join each per-dataset gap with the SPA of the predictive retriever (40.3% NIST23, 17.7% CANOPUS, 13.2% MassSpecGym). Emit a CSV with columns dataset, metric, oracle value, predictive value, gap, SPA, then interpret whether lower SPA tracks larger gaps — here the gaps (~44.4/16.6/9.2 points on Top-1) are largest where SPA is highest, i.e. generation capacity itself caps accuracy when retrieval is near-perfect. The rationale is that the oracle condition bounds what generation can achieve given the correct scaffold, so the difference isolates the retrieval-error penalty.

## Related tools

- **Python** (Computes the per-dataset oracle-minus-predictive gaps for each metric and assembles the joined gap-vs-SPA CSV table)
- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) for Tanimoto similarity used as one of the gap metrics) — https://github.com/HassounLab/MADGEN
- **MADGEN** (Produces the oracle-retriever and predictive-retriever generation outputs being compared; oracle scaffold comes from the lookup table) — https://github.com/HassounLab/MADGEN

## Examples

```
import pandas as pd; df = pd.read_csv('per_condition_metrics.csv'); df['gap'] = df['oracle'] - df['predictive']; df.to_csv('oracle_vs_pred_gap.csv', index=False)
```

## Evaluation signals

- Tabulated oracle and predictive values must match the reproduced per-condition metric tables exactly (e.g. Top-1: oracle 49.0/18.7/10.5 vs predictive 4.6/2.10/1.31 for NIST23/CANOPUS/MassSpecGym)
- Computed gaps must match the expected magnitudes: ~44.4, ~16.6, and ~9.2 percentage points on Top-1 for NIST23, CANOPUS, and MassSpecGym respectively
- SPA join values must equal 40.3% (NIST23), 17.7% (CANOPUS), 13.2% (MassSpecGym)
- The gap-vs-SPA relationship must be qualitatively consistent with the paper's two-retriever evaluation (largest gap on the dataset with highest SPA), verified by expert review
- Output CSV must contain all required columns (dataset, metric, oracle, predictive, gap, SPA) with one row per dataset-per-metric combination

## Limitations

- The SPA-to-gap relationship is qualitative, not a fitted causal model — it indicates scaffold retrieval error constrains generation but does not quantify a functional form
- MSNovelist baseline results are limited to accuracy metrics; similarity and MCES comparisons are unavailable for it (denoted '-')
- Predictive-retriever accuracy is severely constrained by low SPA (17.7%/13.2% on CANOPUS/MassSpecGym), so gaps on those datasets partly reflect ceiling effects of generation rather than purely retriever quality

## Evidence

- [other] SPA and accuracy numbers from source card: [evidence span withheld — anchor status: not_found]
