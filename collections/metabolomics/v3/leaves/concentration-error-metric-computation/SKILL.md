---
name: concentration-error-metric-computation
description: Use when MCF-based concentration reconstruction (mcfNMR setup A/B/C)
  has produced predicted concentrations c_k for a mixture of known composition and
  you must quantify reconstruction accuracy via mean relative error across compounds,
  sweeping assignment radius r to locate the optimum.
when_to_use_negative:
- When only compound identification (containment detection via threshold ϑ) is needed
  and no concentration estimates exist to compare — use the classification/containment
  workflow instead
- When the mixture's true composition is unknown (e.g., natural plasma sample) — no
  ground truth exists to compute relative error
- When concentration accuracy is not the objective, e.g. peak-shift sensitivity or
  library-coverage studies where F1-score is the relevant metric
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3272
tools:
- name: mcfNMR
  role: Produces predicted concentrations c_k via minimum-cost-flow optimization across
    setups A/B/C; result files feed the error computation
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: SciPy
  role: Used to interface HiGHS for the linear optimization that yields flow/concentrations;
    SciPy also supports error metric assembly in Python
- name: HiGHS
  role: Solves the linear optimization of eq 8 underlying flow assignment, whose results
    are the concentration values being evaluated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:9370b3cc742118f535c57dbe5c5269eff92cc011d8df7989aef68a171a99437d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Concentration Error Metric Computation

## Summary

Computes the mean relative error magnitude between MCF-predicted and true compound concentrations for NMR mixtures (e.g., the 34-compound 3 mM mixture), yielding the setup-specific accuracy values reported in the paper (setup A: 0.25 at r=0.15, setup B: 0.30 at r=0.1, setup C: 0.18 at r=0.25).

## When to use

Use when MCF-based concentration reconstruction (mcfNMR setup A/B/C) has produced predicted concentrations c_k for a mixture of known composition and you must quantify reconstruction accuracy via mean relative error across compounds, sweeping assignment radius r to locate the optimum.

## When NOT to use

- When only compound identification (containment detection via threshold ϑ) is needed and no concentration estimates exist to compare — use the classification/containment workflow instead
- When the mixture's true composition is unknown (e.g., natural plasma sample) — no ground truth exists to compute relative error
- When concentration accuracy is not the objective, e.g. peak-shift sensitivity or library-coverage studies where F1-score is the relevant metric

## Inputs

- Predicted concentrations per compound from mcfNMR runs (setup A single-pass, setup B isolated_fit, setup C incremental_fit) at swept assignment radii
- True concentrations of the 34-compound mixture (3 mM each)
- Assignment radius values r (sequence, e.g., 0.1–0.3) tested per setup

## Outputs

- Mean relative error magnitude per (setup, assignment radius) pair
- Optimal assignment radius per setup (A: 0.15; B: 0.1; C: 0.25)
- Comparison table e.g. quantification_results.csv listing setup, r, mean relative error (A:0.25, B:0.30, C:0.18)

## How to apply

1. Load predicted concentrations from mcfNMR result files for each setup: single-pass library fit (setup A), independent compound optimization (isolated_fit=true, setup B), and incremental assignment (incremental_fit=true, setup C). 2. Sweep the assignment radius per setup — setup A peaks at r≈0.15, setup B at r≈0.1, and setup C near r≈0.25. 3. For each run, compare each compound's predicted concentration c_k = alpha_k * c_k_degree against the true value (3 mM for the 34-compound mixture) and compute relative errors, then average their magnitudes across all 34 compounds. 4. Select the assignment radius minimizing mean relative error magnitude per setup and record the triple (setup, optimal r, mean relative error): (A, 0.15, 0.25), (B, 0.1, 0.30), (C, 0.25, 0.18). 5. Assemble results into a table (e.g., quantification_results.csv) to compare semiquantitative accuracy across configurations. Rationale: the paper uses mean relative error magnitude as the primary quantitative-reconstruction metric, demonstrating that incremental assignment (setup C) is most accurate and robust for concentrations up to 34 compounds.

## Related tools

- **mcfNMR** (Produces predicted concentrations c_k via minimum-cost-flow optimization across setups A/B/C; result files feed the error computation) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **SciPy** (Used to interface HiGHS for the linear optimization that yields flow/concentrations; SciPy also supports error metric assembly in Python)
- **HiGHS** (Solves the linear optimization of eq 8 underlying flow assignment, whose results are the concentration values being evaluated)

## Examples

```
python -m mcfnmr -c config_conc.toml  # config: lib=lib.csv, target=target.csv, isolated_fit / incremental_fit per setup, assignment_radius sequence, detection_threshold; then compute mean relative error vs 3 mM truth
```

## Evaluation signals

- Mean relative error magnitude for setup A at r=0.15 equals 0.25
- Mean relative error magnitude for setup B at r=0.1 equals 0.30
- Mean relative error magnitude for setup C at r≈0.25 equals 0.18
- For each setup, error-vs-r curve exhibits a clear minimum at the stated radius; deviation from these values indicates parameter misconfiguration
- All 34 compounds contribute one relative-error term per run; missing/nan entries invalidate the metric

## Limitations

- Quantification accuracy is limited (errors ~0.18–0.30); the paper flags limited quantification accuracy as a current limitation
- Impact of library containing more compounds than the mixture is difficult to predict for quantification
- Sensitivity toward peak shifts distorts concentration estimates
- Diminishing performance for mixtures of increasing complexity
- No definite protocol exists for choosing parameter values (r, cø, ϑ); choice is empirical

## Evidence

- [full_text] Setup-specific optimum radii and error values: [evidence span withheld — anchor status: not_found]
- [abstract] Semiquantitative accuracy scope: "our method retrieves individual compound concentrations with at least semiquantitative accuracy for artificial mixtures with up to 34 compounds"
- [full_text] Detection threshold parameterization: "may thus be obtained by observing the dependency of the number of detections in a blank spectrum with the same noise characteristics on the threshold value"
- [readme] Config file controls setups: "isolated_fit: Boolean flag (true or false) to indicate whether all compounds shall be fitted independently"
- [readme] Config file controls incremental fit: "incremental_fit: Boolean flag (true or false) to indicate whether a multipass incremental fit should be performed. In this case, a sequence of assignment radii must be specified at 'assignment_radius'"
- [readme] Demo dataset source: "several grid spectra of individual compounds and mixtures provided by GeoMetabolomics @ ICBM, University of Oldenburg ([Dataset link](https://zenodo.org/records/14888536))."
