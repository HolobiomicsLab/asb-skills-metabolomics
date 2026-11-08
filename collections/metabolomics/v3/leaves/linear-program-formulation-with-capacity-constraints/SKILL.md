---
name: linear-program-formulation-with-capacity-constraints
description: Use when a target HSQC NMR spectrum must be reconstructed as a union
  of library compound spectra scaled by concentration factors, with joint capacity
  constraints preventing over-assignment of shared peaks.
when_to_use_negative:
- When the mixture contains compounds whose spectral peak pattern is not represented
  in the library — the method cannot complete the reconstruction for unknown constituents.
- 'When spectra have strong peak shifts between library and target: the method is
  explicitly noted as sensitive toward peak shifts, which the local assignment radius
  only partially compensates.'
- When the input is already a fully deconvolved list of compound concentrations —
  the LP reconstruction would be redundant.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: mcfNMR
  role: Reference implementation that constructs the flow network, encodes the four
    feasibility constraints, and invokes the LP solver; provides the 'mcfNMR' and
    'spec2csv' entry points.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Linear optimization backend used to solve the minimum cost flow LP (eq 8)
    that computes the optimal assignment of flow from source to sink nodes.
- name: SciPy
  role: Python interface through which HiGHS is called (scipy.optimize.linprog) for
    the linear optimization.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:0be14d5dafaa447464ab118232a00b947834bc197b96fd2dbac216334808613c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# linear-program-formulation-with-capacity-constraints

## Summary

Formulate NMR mixture reconstruction as a minimum-cost-flow linear program over a network of compound hub nodes, compound-peak nodes, capacity-limited mixture nodes, and an absorption sink, then solve it with the HiGHS solver via the SciPy interface.

## When to use

Use when a target HSQC NMR spectrum must be reconstructed as a union of library compound spectra scaled by concentration factors, with joint capacity constraints preventing over-assignment of shared peaks.

## When NOT to use

- When the mixture contains compounds whose spectral peak pattern is not represented in the library — the method cannot complete the reconstruction for unknown constituents.
- When spectra have strong peak shifts between library and target: the method is explicitly noted as sensitive toward peak shifts, which the local assignment radius only partially compensates.
- When the input is already a fully deconvolved list of compound concentrations — the LP reconstruction would be redundant.

## Inputs

- Library csv file with columns '1H', '13C', optional 'weights', and 'name' (compound ids), optionally gzipped (.csv.gz)
- Target (mixture) spectrum csv file with columns '1H', '13C', optional 'weights', optionally gzipped
- Configuration toml file specifying assignment_radius (float or sequence of floats), detection_threshold, and optionally isolated_fit, incremental_fit, output_dir, and plot flags

## Outputs

- Optimal assignment flows from compound hubs to their peak nodes (saved as .pickle MCF-results)
- Concentration factors alpha_k per compound (fraction of maximal capacity used)
- Detection indicators comparing assignment weight against the detection threshold theta (results as .csv)
- Optional plot of matched compounds overlaid on the target spectrum (svg/png/eps/jpg per gfxformat)

## How to apply

Build a flow network with a source feeding compound hub nodes (each with input capacity equal to a maximal compound amount), compound-peak intermediate nodes, capacity-limited mixture nodes receiving flow from all compounds whose peaks fall within the assignment radius r, and an absorption sink whose per-unit absorption cost c_phi is chosen larger than r so unmatched target capacity is absorbed. Add edges from compound-peak nodes to mixture nodes weighted by the difference between compound peak intensities and target peak intensities, and connect every mixture node to the absorption sink with cost c_phi. Enforce four feasibility constraints: flow conservation at all nodes, capacity limits on compound source flows, capacity limits on mixture nodes (the target total intensity), and the joint capacity constraint that prevents simultaneous over-assignment of multiple compounds onto shared target peaks (disabled when isolated_fit=true). Solve the resulting linear program with scipy.optimize.linprog using the HiGHS backend, then extract per-compound concentration factors alpha_k as the fraction of each compound's maximal capacity that is actually used. When incremental_fit is enabled, repeat the solve over a sequence of assignment radii; otherwise use a single-pass optimization with one float radius.

## Related tools

- **mcfNMR** (Reference implementation that constructs the flow network, encodes the four feasibility constraints, and invokes the LP solver; provides the 'mcfNMR' and 'spec2csv' entry points.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Linear optimization backend used to solve the minimum cost flow LP (eq 8) that computes the optimal assignment of flow from source to sink nodes.)
- **SciPy** (Python interface through which HiGHS is called (scipy.optimize.linprog) for the linear optimization.)

## Examples

```
python -m mcfnmr -c config.toml  # config: lib=lib.csv, target=target.csv, assignment_radius=0.05, detection_threshold=0.1, isolated_fit=false
```

## Evaluation signals

- The LP must solve to optimality under HiGHS with all four feasibility constraints satisfied (flow conservation, compound source capacities, mixture node capacities equal to target total intensity, and the joint capacity constraint).
- Returned assignment flows must jointly respect the maximal target capacity when isolated_fit is disabled.
- Concentration factors alpha_k should be within [0, 1] of each compound's maximal capacity, with semiquantitative accuracy expected for artificial mixtures of up to 34 compounds.
- Assignment weights above the detection threshold theta should correspond to true compound containments; the number of detections in a blank spectrum with the same noise characteristics should remain low as theta varies.
- F1 score on a benchmark set (e.g. the 501-compound MetaboMiner library mapped onto artificial mixtures of 21–27 compounds) should exceed that of MetaboMiner, COLMAR-HSQC, and SMART-Miner by a relative margin of about 15%.

## Limitations

- Reliance on choice of parameters and method variant (assignment radius r, absorption cost c_phi, detection threshold ϑ) — no definite protocol exists to choose these values.
- Sensitivity toward peak shifts, which can shift, change shape, or even disappear depending on the sample matrix and temperature.
- Diminishing performance for mixtures of increasing complexity, and limited quantification accuracy for concentrations.
- Some TextTest runs can fail because output of the subprocess running linear optimization does not synchronize with the output of the main process; such cases should be identified by inspecting the raw output.

## Evidence

- [readme] four feasibility constraints: "isolated_fit: Boolean flag (true or false) to indicate whether all compounds shall be fitted independently. Otherwise, assigned weights must jointly respect the maximal target capacity."
- [discussion] HiGHS solver backend: "for our implementation, we used the SciPy25 interface to HiGHS22"
- [intro] absorption sink cost: "determining whether a peak of a spectrum X appears in Y requires a locally restricted comparison of intensities. We addressed this problem by introducing an assignment radius r"
- [readme] joint capacity constraint: "assigned weights must jointly respect the maximal target capacity"
- [discussion] single pass vs incremental: [evidence span withheld — anchor status: not_found]
