---
name: highs-lp-solver-via-scipy-interface
description: Use when a spectral mixture reconstruction has been cast as a minimum
  cost flow LP on the mcfNMR network (compound hubs, peak nodes, capacity-limited
  mixture nodes, absorption sink) and must be solved to optimality to obtain flow
  assignments and concentration factors.
when_to_use_negative:
- Peak matching is purely local (assignment radius) rather than global EMD-style redistribution,
  so do not use this LP formulation when a global Wasserstein-distance transport over
  the full spectrum without capacity limits is intended.
- The compound library lacks spectra of some mixture constituents, since a complete
  reconstruction requires knowledge of every constituent's peak pattern and unknown
  compounds cannot be recovered by the flow.
- The problem is not a linear program but a nonlinear or combinatorial optimization
  (e.g. direct discrete peak classification), which HiGHS via SciPy cannot handle.
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: HiGHS
  role: Backend linear optimization solver invoked to compute the optimal assignment
    of flow from source to sink nodes (eq 8) in mcfNMR
- name: SciPy
  role: Python interface through which HiGHS is called (scipy.optimize.linprog) for
    the LP underlying the minimum cost flow
- name: mcfNMR
  role: Software that constructs the flow network, formulates the LP, calls HiGHS
    via SciPy, and post-processes flows into alpha_k and detection indicators
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:5e36953905bdb3ed14c67be1d5a602e7f0fc65d1a55703a32e7f2d5fe99ca271
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# highs-lp-solver-via-scipy-interface

## Summary

Solves the mcfNMR minimum cost flow linear program (eq 8) using the HiGHS optimization backend invoked through the SciPy interface, yielding the optimal flow assignment from which compound concentration factors alpha_k and detection indicators are extracted.

## When to use

Use when a spectral mixture reconstruction has been cast as a minimum cost flow LP on the mcfNMR network (compound hubs, peak nodes, capacity-limited mixture nodes, absorption sink) and must be solved to optimality to obtain flow assignments and concentration factors.

## When NOT to use

- Peak matching is purely local (assignment radius) rather than global EMD-style redistribution, so do not use this LP formulation when a global Wasserstein-distance transport over the full spectrum without capacity limits is intended.
- The compound library lacks spectra of some mixture constituents, since a complete reconstruction requires knowledge of every constituent's peak pattern and unknown compounds cannot be recovered by the flow.
- The problem is not a linear program but a nonlinear or combinatorial optimization (e.g. direct discrete peak classification), which HiGHS via SciPy cannot handle.

## Inputs

- Linear program representing the mcfNMR minimum cost flow (eq 8): objective coefficients, constraint matrix, and bounds derived from compound library and target spectrum CSVs (columns '1H', '13C', optional 'weights', plus 'name' for the library)
- Assignment radius r (single float, or sequence of floats for incremental_fit)
- Absorption cost c_phi (set larger than r)
- Detection threshold theta (assignment weight above theta indicates containment)
- isolated_fit and incremental_fit boolean flags from the mcfNMR TOML configuration file

## Outputs

- Optimal flow assignment (flows from compound hubs to peak nodes and to the absorption sink) solving eq 8
- Per-compound concentration factor alpha_k (fraction of maximal compound capacity used)
- Detection indicators per compound (assignment weight versus detection threshold theta)
- mcf-results pickle, results CSV, and optional graphics written to output_dir

## How to apply

Build the flow network from the compound library and target spectrum, then formulate the LP objective as total flow cost (peak-intensity difference costs on compound-peak-to-mixture edges plus per-unit absorption cost c_phi on edges to the absorption sink, chosen larger than the assignment radius r) and impose the four feasibility constraints: flow conservation, compound hub capacity limits, mixture node capacity limits, and the joint capacity constraint preventing over-assignment of multiple compounds onto shared target peaks. Pass this LP to HiGHS via scipy.optimize.linprog, which returns the optimal flows; compute alpha_k for each compound as the fraction of its maximal source capacity actually used. Decision points include the single-pass versus incremental_fit mode, the isolated_fit flag (which relaxes the joint capacity constraint so compounds are fitted independently), and the assignment_radius value, which the article reports can be tuned by observing detections in a blank spectrum. Validate that the LP solves to optimality and that all returned flows jointly respect the maximal target capacity when isolated_fit is disabled.

## Related tools

- **HiGHS** (Backend linear optimization solver invoked to compute the optimal assignment of flow from source to sink nodes (eq 8) in mcfNMR)
- **SciPy** (Python interface through which HiGHS is called (scipy.optimize.linprog) for the LP underlying the minimum cost flow)
- **mcfNMR** (Software that constructs the flow network, formulates the LP, calls HiGHS via SciPy, and post-processes flows into alpha_k and detection indicators) — https://github.com/GeoMetabolomics-ICBM/mcfNMR

## Examples

```
(.mcfnmr_venv)> mcfNMR -c data/user_templates/config_basic.toml
```

## Evaluation signals

- The LP solve via HiGHS terminates with an optimal status (no infeasibility or suboptimal termination reported).
- All four feasibility constraints hold on the returned solution: flow conservation at every node, per-compound hub capacity limits, per-mixture-node capacity limits, and the joint capacity constraint when isolated_fit=false.
- Each alpha_k lies in [0, 1], being the fraction of the compound's maximal capacity actually used.
- Detection indicator for each compound equals (assignment weight > theta), and the number of detections on a blank-spectrum control behaves sensibly as theta is varied.
- Total flow into the absorption sink reflects unmatched target capacity, and aggregate assigned flows do not exceed the maximal target capacity when isolated_fit is disabled.

## Limitations

- No definite answer or protocol exists to choose the assignment radius r, absorption cost c_phi, or detection threshold theta.
- The method is sensitive toward peak shifts, and performance diminishes for mixtures of increasing complexity.
- Quantification accuracy is limited (at best semiquantitative) and the exact impact of a library containing more compounds than the mixture is difficult to predict.
- In the mcfNMR test suite, output of the subprocess running linear optimization may unsynchronize with the main process output, causing rare spurious test failures.

## Evidence

- [discussion] HiGHS is invoked through SciPy for the mcfNMR LP: "for our implementation, we used the SciPy25 interface to HiGHS22"
- [intro] Optimization of eq 8 uses HiGHS: "The optimization of eq 8 is implemented using HiGHS.22"
- [discussion] Linear optimization computes optimal flow assignment: "lies a linear optimization that computes an optimal assignment of flow from source to sink nodes"
- [discussion] Single pass MCF optimization is one flow construction approach: "We explored two approaches for flow construction: a single pass MCF optimization and an incremental assignment"
- [readme] isolated_fit flag semantics from README: "Boolean flag (true or false) to indicate whether all compounds shall be fitted independently. Otherwise, assigned weights must jointly respect the maximal target capacity."
- [readme] Detection threshold semantics from README: "detection_threshold:  Detection threshold (assignment > th indicates containment)"
- [readme] Test suite note on LP subprocess output sync: "some tests may fail because output of a subprocess running linear optimization does not synchronize with the output of the main process"
