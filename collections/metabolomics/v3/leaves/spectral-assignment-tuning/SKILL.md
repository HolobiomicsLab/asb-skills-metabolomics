---
name: spectral-assignment-tuning
description: 'Use when a 2D 1H,13C HSQC mixture-spectrum reconstruction pipeline shows
  parameter sensitivity: you must choose assignment_radius and detection_threshold
  values, compare single-pass vs incremental MCF variants, or validate that F1 scores
  remain stable over a plausible parameter grid.'
when_to_use_negative:
- The spectral data are already peak lists and you only need peak-picking or deconvolution
  (e.g. DEEP picker) rather than MCF assignment-parameter tuning
- You need to choose between distinct MCF method variants (isolated_fit, incremental_fit)
  on biological grounds rather than by scanning (r, theta) — the article states no
  definite protocol for parameter choice exists
- Quantification accuracy of recovered concentrations is the primary goal; this skill
  optimizes identification F1 and offers limited quantification-accuracy guidance
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: mcfNMR
  role: Runs the minimum cost flow spectral assignment for each (r, theta) grid point;
    exposes assignment_radius, detection_threshold, incremental_fit, isolated_fit
    parameters via its TOML config; produces .pickle and .csv result files consumed
    by the tuning analysis.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: SciPy
  role: Pandas/SciPy Python stack used to assemble the per-setup optima and stability
    summaries into a CSV table; mcfNMR itself uses the SciPy interface to HiGHS for
    the underlying linear optimization.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:93434690243dfc979ff34935e62012b052492a7bed394f04442ebee330b81f72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-assignment-tuning

## Summary

Tunes the assignment radius r and detection threshold theta of the minimum cost flow (MCF) NMR spectral-assignment method by scanning parameter combinations and quantifying F1 robustness across MCF setups (single pass vs incremental assignment), identifying stable optima for compound identification on HSQC mixture spectra.

## When to use

Use when a 2D 1H,13C HSQC mixture-spectrum reconstruction pipeline shows parameter sensitivity: you must choose assignment_radius and detection_threshold values, compare single-pass vs incremental MCF variants, or validate that F1 scores remain stable over a plausible parameter grid.

## When NOT to use

- The spectral data are already peak lists and you only need peak-picking or deconvolution (e.g. DEEP picker) rather than MCF assignment-parameter tuning
- You need to choose between distinct MCF method variants (isolated_fit, incremental_fit) on biological grounds rather than by scanning (r, theta) — the article states no definite protocol for parameter choice exists
- Quantification accuracy of recovered concentrations is the primary goal; this skill optimizes identification F1 and offers limited quantification-accuracy guidance

## Inputs

- F1 score grid from a prior mcfNMR scan over (assignment_radius r, detection threshold theta) combinations for setups A, B, C, D
- mcfNMR configuration file (.toml) with assignment_radius and detection_threshold fields
- 2D 1H,13C HSQC library- and target-spectrum CSV files (columns '1H', '13C', 'weights', plus 'name' for library entries)
- Blank spectrum with same noise characteristics for threshold selection

## Outputs

- CSV table of per-setup optimal (r, theta) pairs and average F1 (e.g. C: 0.08/0.003, A: 0.06/0.003, B: 0.05/0.003, D: 0.06/0.005)
- Per-setup F1 robustness summary showing stability across increasing assignment radii
- Detection threshold choice justified by blank-spectrum detection counts vs threshold curve

## How to apply

Construct a parameter grid over assignment radius r and detection threshold theta and compute F1 scores for each MCF setup (A: single pass; B/C/D: incremental assignment variants) using mcfNMR, whose SciPy/HiGHS-based linear optimization solves the minimum cost flow for each (r, theta) pair. Extract the per-setup F1 landscape from the task grid and locate the maximum F1 per setup to obtain optimal (r, theta) pairs — the article reports C: 0.08/0.003, A: 0.06/0.003, B: 0.05/0.003, D: 0.06/0.005. Assess robustness by verifying whether F1 stays flat across a wide range of increasing r values: setup C (incremental assignment) was most robust with the highest average F1 = 0.83 at theta = 0.003 and r = 0.08, followed by A (0.81), B (0.80), and D (0.78). Quantify degradation of each setup as its radius deviates from the optimum (e.g. B's F1 drops as r moves away from 0.05). Use single-pass assignment runs to inform the optimal radius for the incremental setups, and select the detection threshold by observing how many detections a blank spectrum with the same noise characteristics yields as the threshold varies. Write the per-setup optima and stability summaries into a CSV table with pandas for reproducibility.

## Related tools

- **mcfNMR** (Runs the minimum cost flow spectral assignment for each (r, theta) grid point; exposes assignment_radius, detection_threshold, incremental_fit, isolated_fit parameters via its TOML config; produces .pickle and .csv result files consumed by the tuning analysis.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **SciPy** (Pandas/SciPy Python stack used to assemble the per-setup optima and stability summaries into a CSV table; mcfNMR itself uses the SciPy interface to HiGHS for the underlying linear optimization.)

## Examples

```
python -m mcfnmr -c config_grid.toml  # assignment_radius=[0.04,0.05,0.06,0.08], detection_threshold=0.003
```

## Evaluation signals

- The assembled table reproduces the reported optimal (r, theta) per setup: C: 0.08/0.003, A: 0.06/0.003, B: 0.05/0.003, D: 0.06/0.005
- Setup C's F1 landscape is flat and high (average F1 ≈ 0.83) across increasing assignment radii, confirming robustness of the incremental assignment
- Average F1 ranking across setups matches A (0.81) > B (0.80) > D (0.78) besides C (0.83) at their respective optima
- Deviation of B's assignment radius from its optimum (0.05) yields monotonic F1 degradation consistent with its lower robustness
- Threshold-selection check: number of detections in a blank spectrum with the same noise characteristics decreases toward zero as threshold increases, anchoring the chosen theta

## Limitations

- No definite answer or protocol exists for choosing r, theta (and the absorption cost parameter); tuning remains largely empirical and dataset-specific
- The reported optima are tied to the evaluated artificial mixtures (21–27 compounds for MetaboMiner data; up to 34 compounds for the in-house 34-compound library) and may not transfer to more complex or differently shifted spectra
- mcfNMR methods are sensitive to peak shifts between library and target spectra, and diminishing performance for mixtures of increasing complexity limits generalization of tuned parameters
- The 'aggregate_runs' test suite can take several hours on moderately fast CPUs, so full parameter sweeps can be time-consuming

## Evidence

- [results] Across scanned (r, theta) combinations, setup C (incremental assignment) is the most robust, achieving the highest average F1 = 0.83 at theta = 0.003 and r = 0.08: [evidence span withheld — anchor status: not_found]
- [discussion] may thus be obtained by observing the dependency of the number of detections in a blank spectrum with the same noise characteristics on the threshold value: "may thus be obtained by observing the dependency of the number of detections in a blank spectrum with the same noise characteristics on the threshold value"
- [discussion] To date, we have no definite answer or protocol to choose these values: "To date, we have no definite answer or protocol to choose these values"
- [readme] assignment_radius: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true).: "assignment_radius: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true)."
- [discussion] for our implementation, we used the SciPy25 interface to HiGHS22: "for our implementation, we used the SciPy25 interface to HiGHS22"
