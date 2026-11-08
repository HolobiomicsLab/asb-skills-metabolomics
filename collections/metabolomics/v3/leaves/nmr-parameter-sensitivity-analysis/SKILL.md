---
name: nmr-parameter-sensitivity-analysis
description: Use when an mcfNMR run has produced F1 scores over scanned (r, theta)
  combinations for multiple MCF setups and you need to determine parameter robustness
  and optimal (r, theta) settings per setup.
when_to_use_negative:
- The input is raw 1H,13C HSQC peak lists or target spectra rather than already-computed
  F1 scores over a parameter grid — run the sensitivity sweep itself first.
- Only a single mcfNMR configuration (one r, one theta) was run, so no (r, theta)
  landscape exists to analyze.
- The goal is absolute performance comparison against external methods (MetaboMiner,
  COLMAR-HSQC, SMART-Miner) rather than internal parameter tuning of mcfNMR setups.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mcfNMR
  role: Produces the F1 grids over assignment_radius and detection_threshold per setup
    (single-pass, incremental_fit variants) that this skill analyzes
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: SciPy
  role: Used alongside pandas to process and summarize the per-setup F1 landscapes
    (optimization/lookup of optima, degradation quantification)
- name: pandas
  role: Assembles per-setup optima and stability summaries into the final CSV output
    table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:7dd3436217e18a75a04e3e1317fca110edf4e77f9b61cec4e973207b88011bb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - SciPy
  - mcfNMR
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# nmr-parameter-sensitivity-analysis

## Summary

Scans the assignment radius (r) and detection threshold (theta) grids of mcfNMR setups (single-pass A/B, incremental C, flow-based D) to map F1 robustness across artificial 2D 1H,13C HSQC mixture spectra and identify per-setup (r, theta) optima.

## When to use

Use when an mcfNMR run has produced F1 scores over scanned (r, theta) combinations for multiple MCF setups and you need to determine parameter robustness and optimal (r, theta) settings per setup.

## When NOT to use

- The input is raw 1H,13C HSQC peak lists or target spectra rather than already-computed F1 scores over a parameter grid — run the sensitivity sweep itself first.
- Only a single mcfNMR configuration (one r, one theta) was run, so no (r, theta) landscape exists to analyze.
- The goal is absolute performance comparison against external methods (MetaboMiner, COLMAR-HSQC, SMART-Miner) rather than internal parameter tuning of mcfNMR setups.

## Inputs

- Per-setup F1 score grid over scanned assignment radius (r) and detection threshold (theta) combinations, as produced by mcfNMR configuration runs (setup A: single-pass fit; setup B: single-pass with absorption; setup C: incremental_fit=true; setup D: flow-based)

## Outputs

- CSV table of per-setup optimal (r, theta) pairs with average F1 values
- Robustness/stability summary describing F1 landscape flatness and degradation away from each optimum

## How to apply

Load the F1 grid produced by sweeping assignment_radius and detection_threshold values, then isolate the per-setup F1 landscapes for setups A, B, C, and D. Use pandas to assemble the landscape into a table and SciPy/numpy routines (or plain max-lookup) to locate the (r, theta) pair that maximizes F1 for each setup. Verify robustness by confirming that the winning setup's F1 remains flat and high across a range of increasing assignment radii rather than spiking at a single value — in the source study, setup C (incremental assignment) achieved the highest average F1 = 0.83 at theta = 0.003 and r = 0.08, followed by A (0.81, r=0.06), B (0.80, r=0.05), and D (0.78, r=0.06/theta=0.005). Quantify degradation of each non-winner's F1 as its radius deviates from its optimum (e.g. setup B's drop when r moves away from 0.05). Record the final per-setup optima and stability summary as a CSV for downstream parameter-choice decisions.

## Related tools

- **mcfNMR** (Produces the F1 grids over assignment_radius and detection_threshold per setup (single-pass, incremental_fit variants) that this skill analyzes) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **SciPy** (Used alongside pandas to process and summarize the per-setup F1 landscapes (optimization/lookup of optima, degradation quantification))
- **pandas** (Assembles per-setup optima and stability summaries into the final CSV output table)

## Examples

```
mcfNMR -c config.toml  # with assignment_radius=[0.05,0.06,0.08] and detection_threshold=0.003 swept across setups; then pd.read_csv(f1_grid.csv).groupby('setup').agg([('r','idxmax'),('theta','idxmax'),('f1','max')])
```

## Evaluation signals

- The extracted per-setup optima reproduce the reported values: setup C at r=0.08/theta=0.003 (F1=0.83), A at r=0.06/theta=0.003 (0.81), B at r=0.05/theta=0.003 (0.80), D at r=0.06/theta=0.005 (0.78).
- Setup C's F1 profile shows a flat, high plateau across increasing assignment radii rather than a single sharp peak.
- Ranking by average F1 matches the expected ordering C > A > B > D.
- Quantified degradation of setup B's F1 as r deviates from 0.05 is captured in the stability summary.
- Output CSV parses with pandas and contains one optimal (r, theta) row per MCF setup.

## Limitations

- The article states there is no definite answer or protocol to choose these parameter values — optima found here are empirical, from grid sweeps on artificial mixtures.
- The optimal detection threshold is interlinked with blank-spectrum detections; a threshold too low admits noise hits, so the scan should ideally include a blank-spectrum check.
- Results were derived from artificial mixtures of 21–27 compounds and a 34-compound in-house library; performance diminishes for mixtures of increasing complexity.
- Sensitivity toward peak shifts in the target spectrum affects the whole MCF approach and may shift the optimal assignment radius.

## Evidence

- [results] Across scanned (r, theta) combinations, setup C (incremental assignment) is the most robust, achieving the highest average F1 = 0.83 at theta = 0.003 and r = 0.08, followed by A (0.81), B (0.80), and D (0.78).: [evidence span withheld — anchor status: not_found]
- [discussion] No definite answer or protocol to choose parameter values for assignment radius r, absorption cost cø, and detection threshold ϑ: "To date, we have no definite answer or protocol to choose these values"
- [readme] assignment_radius: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true).: "`assignment_radius`: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true)."
- [discussion] testing single pass assignment conveys important information about the optimal assignment radius for all setups: "testing single pass assignment conveys important information about the optimal assignment radius for all setups"
- [readme] detection_threshold: Detection threshold (assignment > th indicates containment).: "`detection_threshold`: Detection threshold (assignment > th indicates containment)."
