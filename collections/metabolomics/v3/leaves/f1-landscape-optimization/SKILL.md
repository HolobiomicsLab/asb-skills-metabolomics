---
name: f1-landscape-optimization
description: 'Use when tuning mcfNMR runs on artificial 1H,13C HSQC mixtures: an F1
  grid over assignment radius (r) and absorption cost/detection threshold (θ) values
  already exists (from a prior sweep) and you must identify per-setup optima, rank
  setups by robustness, or quantify F1 degradation as r deviates.'
when_to_use_negative:
- You only have a single mcfNMR run at one fixed (r, θ) combination — no landscape
  exists to optimize over; rerun the sweep first.
- The input is a raw grid spectrum or peak list (USCF, .txt.gz, .csv) rather than
  already-computed F1 scores — this skill assumes the identification run and F1 computation
  have already been done.
- You need to select parameters for quantification accuracy rather than detection/identification
  F1 — the article reports limited quantification accuracy as a separate limitation
  and this skill optimizes only the classification metric.
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0091
tools:
- name: mcfNMR
  role: 'Generates the F1 grid being optimized: each configuration file sets ''assignment_radius''
    (single float for single-pass, sequence of floats for incremental_fit=true) and
    ''detection_threshold'', producing mcf-results (.pickle) and results (.csv) per
    scanned (r, θ) combination.'
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: SciPy
  role: Used via the SciPy interface to HiGHS for the underlying linear optimization
    of the minimum cost flow; the article's reference implementation environment.
- name: pandas
  role: Assembles the per-setup optima and stability summaries into the final CSV
    table (per the task workflow).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:d396afe4d505629cd6071ee4894405b0931d12bd5e438fe2421551f696e79f43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - SciPy
  - mcfNMR
  - pandas
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# f1-landscape-optimization

## Summary

Scans the joint grid of assignment radius r and detection threshold θ for each mcfNMR setup (A–D) and extracts the (r, θ) pair maximizing average F1 across artificial HSQC mixtures, while mapping each setup's robustness landscape.

## When to use

Use when tuning mcfNMR runs on artificial 1H,13C HSQC mixtures: an F1 grid over assignment radius (r) and absorption cost/detection threshold (θ) values already exists (from a prior sweep) and you must identify per-setup optima, rank setups by robustness, or quantify F1 degradation as r deviates from its optimum.

## When NOT to use

- You only have a single mcfNMR run at one fixed (r, θ) combination — no landscape exists to optimize over; rerun the sweep first.
- The input is a raw grid spectrum or peak list (USCF, .txt.gz, .csv) rather than already-computed F1 scores — this skill assumes the identification run and F1 computation have already been done.
- You need to select parameters for quantification accuracy rather than detection/identification F1 — the article reports limited quantification accuracy as a separate limitation and this skill optimizes only the classification metric.

## Inputs

- F1 score grid (CSV or table) over scanned (assignment radius r, absorption cost/detection threshold θ) combinations, one row per setup (A–D) and per artificial mixture
- Setup labels for the four mcfNMR variants (A: single-pass isolated, B, C: incremental assignment, D: peak-list based)
- Artificial mixture ground-truth compound identities (e.g. MetaboMiner N925/N987/N988 mixtures with 21–27 compounds) used to compute F1

## Outputs

- Per-setup optimal (r, θ) pair with its maximum average F1 (expected: C 0.08/0.003, A 0.06/0.003, B 0.05/0.003, D 0.06/0.005)
- Robustness/stability summary per setup (flat vs. sensitive F1 landscape across increasing r)
- Quantified F1 degradation for setup B as assignment radius deviates from 0.05
- CSV table (via pandas) of per-setup optima and stability metrics

## How to apply

Load the F1 grid produced by the mcfNMR parameter sweep (computed for combinations of assignment radius r and absorption cost θ), isolate the per-setup (A, B, C, D) landscapes, and locate the maximum F1 in each to read off the optimal (r, θ) pair. Expected optima are C: 0.08/0.003, A: 0.06/0.003, B: 0.05/0.003, D: 0.06/0.005; setup C (incremental assignment) shows the highest and flattest landscape with average F1 = 0.83 at θ = 0.003, r = 0.08, followed by A (0.81), B (0.80), and D (0.78). Verify robustness by checking that setup C's F1 stays high across increasing assignment radius values, then quantify how setup B's F1 degrades as r moves away from its optimum 0.05. Assemble the per-setup optima and stability summaries into a CSV table with pandas; the article notes no definite protocol exists for choosing r, θ, and ϑ, which is exactly why this empirical landscape scan is needed.

## Related tools

- **mcfNMR** (Generates the F1 grid being optimized: each configuration file sets 'assignment_radius' (single float for single-pass, sequence of floats for incremental_fit=true) and 'detection_threshold', producing mcf-results (.pickle) and results (.csv) per scanned (r, θ) combination.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **SciPy** (Used via the SciPy interface to HiGHS for the underlying linear optimization of the minimum cost flow; the article's reference implementation environment.)
- **pandas** (Assembles the per-setup optima and stability summaries into the final CSV table (per the task workflow).)

## Examples

```
mcfNMR -c config.toml  # config sets assignment_radius=[0.05,0.06,0.08], detection_threshold=0.003, incremental_fit=true; then pandas reads results .csv to extract per-setup F1 maxima
```

## Evaluation signals

- The assembled CSV table must reproduce the reported per-setup optima exactly: setup C (r=0.08, θ=0.003), A (r=0.06, θ=0.003), B (r=0.05, θ=0.003), D (r=0.06, θ=0.005).
- Average F1 ranking must match A=0.81, B=0.80, C=0.83, D=0.78, with setup C achieving the highest value at θ=0.003 and r=0.08.
- Setup C's F1 landscape must remain flat and high across a wide range of increasing assignment radius values, confirming the 'most robust' claim.
- Setup B's F1 must visibly degrade as its assignment radius deviates from 0.05, while C's does not.
- Output CSV must contain one optimum row per setup (A–D) with both r and θ columns and an F1 column; missing any setup indicates the landscape extraction failed.

## Limitations

- The article states there is 'no definite answer or protocol to choose these values' for assignment radius r, absorption cost cø, and detection threshold ϑ — the landscape scan is empirical and mixture-set specific, not a validated universal protocol.
- Performance diminishes for mixtures of increasing complexity, so optima tuned on 21–27 compound artificial mixtures may not transfer to denser samples.
- Parameter choices interact with the method variant (single-pass vs. incremental assignment), and reliance on parameter/variant choice is listed as a limitation of the underlying method.
- Optimization targets detection F1; the article separately notes limited quantification accuracy, so optimal F1 settings may not be optimal for concentration reconstruction.

## Evidence

- [results] optimal average F1 = 0.83 at theta = 0.003 and r = 0.08: [evidence span withheld — anchor status: not_found]
- [discussion] no definite answer or protocol to choose these values: "To date, we have no definite answer or protocol to choose these values"
- [readme] assignment_radius: Single float or sequence of floats: "assignment_radius: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true)."
- [readme] detection_threshold: Detection threshold (assignment > th indicates containment): "detection_threshold:  Detection threshold (assignment > th indicates containment)."
- [discussion] diminishing performance for mixtures of increasing complexity: "diminishing performance for mixtures of increasing complexity"
