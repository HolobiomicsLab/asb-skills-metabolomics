---
name: lcmsms-run-identification-accuracy-evaluation
description: Use when you have per-spectrum scored candidate molecule lists from an
  LC-MS/MS run plus a retention-order predictor, and you need to measure whether integrating
  the two improves the percentage of correctly identified molecular structures.
when_to_use_negative:
- You only have a single MS/MS spectrum to identify (no complete LC-MS/MS run with
  retention ordering across spectra) — the graph-based joint identification needs
  multiple ordered layers
- You lack a trained retention-order predictor (RankSVM w) and only have MS/MS scores
  — then D=0 recovers the plain IOKR baseline and adds nothing
- Molecules have retention times too small to interact with the chromatographic system
  — these were removed during preprocessing and would corrupt the order graph
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: IOKR
  role: Produces the per-spectrum scored candidate molecule lists that are combined
    with retention-order predictions
- name: scikit-learn (sklearn)
  role: Provides the RankSVM order predictor used to score rank conflicts in the edge
    weights
- name: networkx
  role: Supports construction/traversal of the layered directed graph for shortest-path
    search
- name: numpy
  role: Numerical implementation of the DP forward pass and hinge penalty computations
- name: aalto-ics-kepaco/retention_order_prediction
  role: Hosts evaluation_scenarios_main.py which runs the all_on_one (LTSO) scenario
    and writes identification_accuracy_vs_D.csv
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:f4bbcecf6d2d2aa842e9311a68bcf1d5aa122b2e13ccfd85df8e24444f570c0c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IOKR
  - Python and
  - R
  - aalto-ics-kepaco/retention_order_prediction
  - joblib
  - networkx
  - numpy
  - pandas
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# lcmsms-run-identification-accuracy-evaluation

## Summary

Evaluates metabolite identification accuracy over a complete LC-MS/MS run by combining IOKR MS/MS candidate scores with predicted retention orders via a dynamic-programming shortest-path search, sweeping the trade-off constant D to quantify improvement over the IOKR-only baseline.

## When to use

Use when you have per-spectrum scored candidate molecule lists from an LC-MS/MS run plus a retention-order predictor, and you need to measure whether integrating the two improves the percentage of correctly identified molecular structures.

## When NOT to use

- You only have a single MS/MS spectrum to identify (no complete LC-MS/MS run with retention ordering across spectra) — the graph-based joint identification needs multiple ordered layers
- You lack a trained retention-order predictor (RankSVM w) and only have MS/MS scores — then D=0 recovers the plain IOKR baseline and adds nothing
- Molecules have retention times too small to interact with the chromatographic system — these were removed during preprocessing and would corrupt the order graph

## Inputs

- Per-spectrum scored candidate molecule lists produced by IOKR (spectral input kernel, molecular output kernel)
- RankSVM weight vector w trained on retention times (Target, Others, or Others&target variants)
- Molecular feature map phi from MACCS counting fingerprints (minmax kernel)
- Observed retention times and labels y(r,s) for each measured MS/MS spectrum
- Experiment configuration file, e.g. results/raw/PredRet/v2/config.json

## Outputs

- identification_accuracy_vs_D.csv: average percentage of correctly identified molecular structures per D value and training variant
- Optimal sequence of candidate molecular structures listed in retention order per run

## How to apply

Build a layered directed graph G with one layer per measured MS/MS spectrum sorted by increasing retention time and candidates within a layer sorted by decreasing IOKR score, adding directed edges between every node pair in consecutive layers. Assign each edge the weight d = -y(r,s) + D * max(0, wT(phi(m(i,j)) - phi(m(r,s)))), where the hinge term penalizes only rank conflicts with the observed retention-time order and is zero when the observed retention times of layers i and r are equal; molecular features phi are MACCS counting fingerprints with a minmax kernel. Run the Bellman-style dynamic-programming forward pass (complexity O(NK^2), N layers, K maximum layer size) to retrieve the optimal sequence of candidate structures in retention order. Sweep D from d_lower=0 to d_upper=16, treating D=0 as the IOKR-only identification baseline, and repeat for the Target/Others/Others&target training variants of the all_on_one (LTSO) scenario using evaluation_scenarios_main.py. For each D and training variant, compute the average percentage of correctly identified molecular structures across the run and write identification_accuracy_vs_D.csv; the D=0 result must match the IOKR-only baseline.

## Related tools

- **IOKR** (Produces the per-spectrum scored candidate molecule lists that are combined with retention-order predictions)
- **scikit-learn (sklearn)** (Provides the RankSVM order predictor used to score rank conflicts in the edge weights)
- **networkx** (Supports construction/traversal of the layered directed graph for shortest-path search)
- **numpy** (Numerical implementation of the DP forward pass and hinge penalty computations)
- **aalto-ics-kepaco/retention_order_prediction** (Hosts evaluation_scenarios_main.py which runs the all_on_one (LTSO) scenario and writes identification_accuracy_vs_D.csv) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
python src/evaluation_scenarios_main.py ranksvm all_on_one 10 0 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- The D=0 configuration must reproduce the IOKR-only identification baseline (no order penalty applied)
- Results must reproduce the reported qualitative trend: combining retention-order predictions with MS/MS scores improves average percentage of correctly identified molecular structures over the IOKR-only baseline, with improvement up to 24.8% at D=0.0075 for the Others&target (LTSO) RankSVM variant
- D sweep covers the repository's evaluated range d_lower=0 to d_upper=16 with intermediate granularity capturing D=0.0075
- All three training variants (Target, Others, Others&target) of the all_on_one LTSO scenario are evaluated and stored under results/PredRet/v2/final/..._type=order_graph
- Output CSV has one accuracy value per (D, training-variant) pair computed across the complete LC-MS/MS run

## Limitations

- The reported improvement (up to 24.8% at D=0.0075) requires RankSVM trained on retention times from both other systems and the target system (Others & target); performance with target-only training may differ
- The D parameter is only qualitatively characterized in the paper's discussion — quantitative accuracy values for each D are referenced via a figure, so exact values must be recomputed
- The hinge penalty is zero when observed retention times of consecutive layers are equal, so identification among co-eluting compounds is driven by IOKR scores alone

## Evidence

- [abstract] Retention order prediction combined with MS/MS-based scores improves metabolite identifications when analyzing a complete LC-MS/MS run.: [evidence span withheld — anchor status: not_found]
- [methods] DP complexity: [evidence span withheld — anchor status: not_found]
