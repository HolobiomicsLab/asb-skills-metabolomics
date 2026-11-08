---
name: cross-tool-prediction-performance-comparison
description: Use when a new in silico reactant-prediction or site-of-metabolism tool
  must be benchmarked against CypReact, ADMET Predictor, or SMARTCyp on the 1632-molecule
  CYP450 reactant dataset using per-enzyme AUROC.
when_to_use_negative:
- The goal is predicting a single site of metabolism (SoM) per molecule rather than
  binary reactant classification per CYP450 enzyme — SMARTCyp's original mode, not
  this reactant-prediction comparison.
- ADMET Predictor must be computed de novo but no license is available — it is commercial,
  so only its reported average AUROC of 0.75 can be recorded for comparison.
- The molecule set differs from the 1632-molecule benchmark and no cross-validation
  protocol is being reproduced — AUROC averages are then not comparable to the published
  0.83–0.92 / 0.75 / 0.53 figures.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: CypReact
  role: Reference tool whose cross-validation AUROC (0.83–0.92 on 1632 molecules)
    is the upper baseline for the comparison
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: ADMET Predictor
  role: Commercial baseline tool; its reported average AUROC of 0.75 is recorded for
    comparison since it cannot be freely executed
- name: SMARTCyp
  role: Baseline tool via its reactant-predicting extension; re-run over the benchmark
    to reproduce its reported average AUROC of 0.53
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:2cb34ffc2e74177ea5c01f34d594b909d29f80bdc84d9bbaaf7f8123351eb447
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - CypReact
  - SMARTCyp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-tool-prediction-performance-comparison

## Summary

Benchmarks reactant-prediction software for the nine human CYP450 enzymes by computing per-enzyme AUROCs on the 1632-molecule benchmark and comparing tool averages (e.g. CypReact 0.83–0.92 vs ADMET Predictor 0.75 vs SMARTCyp 0.53). It establishes whether a new tool significantly outperforms published baselines.

## When to use

Use when a new in silico reactant-prediction or site-of-metabolism tool must be benchmarked against CypReact, ADMET Predictor, or SMARTCyp on the 1632-molecule CYP450 reactant dataset using per-enzyme AUROC.

## When NOT to use

- The goal is predicting a single site of metabolism (SoM) per molecule rather than binary reactant classification per CYP450 enzyme — SMARTCyp's original mode, not this reactant-prediction comparison.
- ADMET Predictor must be computed de novo but no license is available — it is commercial, so only its reported average AUROC of 0.75 can be recorded for comparison.
- The molecule set differs from the 1632-molecule benchmark and no cross-validation protocol is being reproduced — AUROC averages are then not comparable to the published 0.83–0.92 / 0.75 / 0.53 figures.

## Inputs

- 1632-molecule reactant benchmark dataset (SMILES strings or SDF records) with binary reactant labels per CYP450 enzyme
- Per molecule–enzyme pair reactant scores from each tool under comparison (CypReact, ADMET Predictor, reactant-predicting SMARTCyp)
- Reported baseline AUROC values for reference tools

## Outputs

- Per-enzyme AUROC values (nine CYP450s) for each evaluated tool
- Average AUROC per tool compared against reported baselines (0.83–0.92 CypReact, 0.75 ADMET Predictor, 0.53 SMARTCyp)
- Results table, e.g. smartcyp_auroc_results.csv, written for expert review

## How to apply

Load the 1632-molecule reactant dataset (SMILES or SDF records) with binary reactant labels for each of the nine CYP450 enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4). Run every candidate tool over all molecule–enzyme pairs, mirroring CypReact's cross-validation protocol, to produce a reactant score per pair. Compute AUROC separately for each of the nine enzymes, then average the nine values. Compare against the reference baselines: CypReact cross-validation AUROC 0.83–0.92, ADMET Predictor average AUROC 0.75, and the reactant-predicting extension of SMARTCyp average AUROC 0.53; record the reported values alongside computed ones since ADMET Predictor is commercial and cannot be freely executed. Validate by checking that a re-run reproduces the published average (e.g. SMARTCyp ≈ 0.53) and that conclusions about tool superiority match the reported significance ordering.

## Related tools

- **CypReact** (Reference tool whose cross-validation AUROC (0.83–0.92 on 1632 molecules) is the upper baseline for the comparison) — https://bitbucket.org/Leon_Ti/cypreact
- **ADMET Predictor** (Commercial baseline tool; its reported average AUROC of 0.75 is recorded for comparison since it cannot be freely executed)
- **SMARTCyp** (Baseline tool via its reactant-predicting extension; re-run over the benchmark to reproduce its reported average AUROC of 0.53)

## Evaluation signals

- Recomputed SMARTCyp average AUROC matches the reported value of 0.53
- Computed CypReact AUROC falls within the reported cross-validation range of 0.83–0.92 on the 1632-molecule set
- Nine per-enzyme AUROC values (one per CYP1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) are present and averaged to the reported figure
- Performance ordering of tools (CypReact > ADMET Predictor 0.75 > SMARTCyp 0.53) is consistent with the published finding
- On a held-out previously unseen molecule set, the new tool still significantly surpasses SMARTCyp and ADMET Predictor

## Limitations

- ADMET Predictor is commercial and cannot be freely executed, so its 0.75 average AUROC can only be recorded from literature rather than recomputed.
- Average AUROC values are only comparable when the same 1632-molecule dataset and cross-validation evaluation protocol are used.
- The reactant-predicting extension of SMARTCyp is a derived mode of the original site-of-metabolism tool, and its low average AUROC (0.53) reflects that adaptation rather than SMARTCyp's native SoM performance.

## Evidence

- [abstract] abstract: "Tests of CypReact, conducted over a data set of 1632 molecules (each considered a "plausible" reactant) show that it is very effective"
