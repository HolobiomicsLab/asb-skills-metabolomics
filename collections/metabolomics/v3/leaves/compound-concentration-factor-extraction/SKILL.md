---
name: compound-concentration-factor-extraction
description: Use when a minimum cost flow assignment over an HSQC peak network has
  been solved (e.g. with HiGHS via SciPy) and per-compound concentrations or detection
  indicators are needed from the resulting flows.
when_to_use_negative:
- When the LP has not been solved or the flow assignment is infeasible — αk is only
  defined from a feasible, optimal flow satisfying all four constraints.
- 'When the user expects exact quantification: the method provides at best semiquantitative
  accuracy, with limited quantification accuracy explicitly listed as a current limitation.'
- When a compound's spectral peak pattern is not in the library — a complete reconstruction
  requires knowledge of every constituent's peak pattern.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: mcfNMR
  role: Provides the minimum cost flow network and LP solution from which the concentration
    factors αk are extracted; the main command reports assignment weights per compound
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Linear programming backend solving the min-cost-flow problem whose optimal
    flows define αk
- name: SciPy
  role: Python interface used to call the HiGHS solver (scipy.optimize.linprog) for
    the LP underlying the flow assignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:a284ac614591926bdb9e5aa69bd2a9a10761b88760f199c51fa1ef3f32669cee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# compound-concentration-factor-extraction

## Summary

Extract per-compound concentration factors αk from the optimal minimum cost flow assignment computed by mcfNMR, defined as the fraction of each compound's maximal source capacity actually used in the flow. This turns the LP solution into semiquantitative compound concentrations for 2D ¹H,¹³C HSQC mixture spectra.

## When to use

Use when a minimum cost flow assignment over an HSQC peak network has been solved (e.g. with HiGHS via SciPy) and per-compound concentrations or detection indicators are needed from the resulting flows.

## When NOT to use

- When the LP has not been solved or the flow assignment is infeasible — αk is only defined from a feasible, optimal flow satisfying all four constraints.
- When the user expects exact quantification: the method provides at best semiquantitative accuracy, with limited quantification accuracy explicitly listed as a current limitation.
- When a compound's spectral peak pattern is not in the library — a complete reconstruction requires knowledge of every constituent's peak pattern.

## Inputs

- Solved minimum cost flow assignment from the mcfNMR LP (flow values from compound hubs to compound-peak nodes), e.g. stored in the mcf-results .pickle file
- Compound library csv (columns 'name', '1H', '13C', optionally 'weights') used to construct the network, providing each compound's maximal amount as hub capacity
- Target spectrum csv (columns '1H', '13C', optionally 'weights') defining mixture-node capacities
- Detection threshold θ used to convert assignment weights into containment indicators

## Outputs

- Concentration factor αk for each compound in the library (fraction of maximal capacity used)
- Assignment flows from compound hubs to their peak nodes
- Detection indicators (assignment weight compared with detection threshold θ)
- Result files: mcf-results (.pickle), results table (.csv), and optional graphics (e.g. svg/png/eps)

## How to apply

After the LP is solved to optimality with scipy.optimize.linprog and the HiGHS backend, read the optimal flows from each compound's source hub to its compound-peak nodes. For each compound k, compute the concentration factor αk as the fraction of its maximal compound amount (the input capacity on the compound hub node) that is actually assigned as flow; αk is exactly the 'assigned weight' used against the detection threshold θ to indicate containment. Verify that all four feasibility constraints still hold — flow conservation, compound source capacities, mixture-node capacity (target total intensity), and the joint capacity constraint that prevents simultaneous over-assignment onto shared target peaks — especially when isolated_fit is disabled, so that assigned weights jointly respect the maximal target capacity. Return the assignment flows, the αk values, and the detection indicators (assignment weight vs θ) together with the mcf-results pickle and csv outputs. Note that αk is intended as at least semiquantitative accuracy for artificial mixtures with up to 34 compounds, not exact quantification.

## Related tools

- **mcfNMR** (Provides the minimum cost flow network and LP solution from which the concentration factors αk are extracted; the main command reports assignment weights per compound) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Linear programming backend solving the min-cost-flow problem whose optimal flows define αk)
- **SciPy** (Python interface used to call the HiGHS solver (scipy.optimize.linprog) for the LP underlying the flow assignment)

## Examples

```
mcfNMR -c config.toml  # then inspect the .pickle/.csv results for per-compound assignment weights and αk
```

## Evaluation signals

- The LP solves to optimality under HiGHS with all four feasibility constraints satisfied (flow conservation, compound source capacities, mixture-node capacity, joint capacity).
- Every αk lies in [0, 1], i.e. no compound uses more flow than its maximal hub capacity.
- When isolated_fit is false, the sum of assigned weights over compounds must not exceed the maximal target capacity on each mixture node.
- Detection indicators (assignment weight vs θ) match the number of detections expected from the chosen threshold; a decreasing threshold should not increase detections in a blank spectrum with the same noise characteristics.
- Reported αk values reproduce the semiquantitative retrieval expected on artificial mixtures of up to 34 compounds when rerun on the same library and target.

## Limitations

- Limited quantification accuracy: αk is semiquantitative at best.
- Diminishing performance for mixtures of increasing complexity and reliance on choice of parameters (assignment radius r, absorption cost cφ, detection threshold ϑ) and method variant.
- Sensitivity toward peak shifts: peaks may shift, change shape, or disappear depending on sample matrix and temperature, distorting the flows that define αk.
- Exact impact of the library containing more compounds than the mixture is difficult to predict for quantification.

## Evidence

- [readme] mcfNMR README on configuration and output files: "Output directory for the result files (mcf-results with extension '.pickle', results with extension '.csv' and graphics with user-specified format `gfxformat`)."
- [readme] README on isolated_fit semantics: "isolated_fit`: Boolean flag (true or false) to indicate whether all compounds shall be fitted independently. Otherwise, assigned
weights must jointly respect the maximal target capacity."
- [readme] README on detection threshold: "detection_threshold`:  Detection threshold (assignment > th indicates containment)."
- [abstract] Article abstract on quantification: "our method retrieves individual compound concentrations with at least semiquantitative accuracy for artificial mixtures with up to 34 compounds"
- [discussion] Article discussion on current limitations: "(iv) limited quantification accuracy"
