---
name: minimum-cost-flow-network-construction
description: Use when reconstructing a target 2D 1H,13C HSQC NMR mixture spectrum
  as a weighted union of library compound spectra, especially when peak overlaps and
  inter-compound dependencies must be handled jointly rather than fitted per compound.
when_to_use_negative:
- The mixture may contain compounds whose spectral peak pattern is not known — a complete
  reconstruction requires knowledge of every constituent's peak pattern
- Peak shifts between library and target are large relative to the assignment radius
  r, since the method is explicitly noted as sensitive toward peak shifts
- The task requires a definitive protocol for choosing r, c_phi, or theta — the paper
  states there is 'no definite answer or protocol to choose these values'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3420
tools:
- name: mcfNMR
  role: Reference implementation that builds the flow network, runs the LP, and emits
    assignment flows, concentration factors, and detection indicators from library/target
    csv files
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: LP solver backend used to optimize the minimum cost flow objective (eq 8)
    through the SciPy interface
- name: SciPy
  role: Python interface through which HiGHS is invoked (scipy.optimize.linprog) for
    the linear optimization of flow assignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:faaca4568ae079c9d6eef37132272df6ce7de03711efbdff602d732624a3f4cb
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
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# minimum-cost-flow-network-construction

## Summary

Constructs a minimum cost flow network over library compound HSQC peaks and mixture target peaks to identify and quantify compounds in an NMR mixture via the Earth Mover's Distance analogue, solved as a linear program with HiGHS.

## When to use

Use when reconstructing a target 2D 1H,13C HSQC NMR mixture spectrum as a weighted union of library compound spectra, especially when peak overlaps and inter-compound dependencies must be handled jointly rather than fitted per compound.

## When NOT to use

- The mixture may contain compounds whose spectral peak pattern is not known — a complete reconstruction requires knowledge of every constituent's peak pattern
- Peak shifts between library and target are large relative to the assignment radius r, since the method is explicitly noted as sensitive toward peak shifts
- The task requires a definitive protocol for choosing r, c_phi, or theta — the paper states there is 'no definite answer or protocol to choose these values'

## Inputs

- Library csv file with columns 'name', '1H', '13C', and optionally 'weights' giving peak coordinates and intensities per compound (.csv or .csv.gz)
- Target (mixture) spectrum csv file with columns '1H', '13C', and optionally 'weights' (.csv or .csv.gz)
- Configuration toml file specifying assignment_radius, detection_threshold, and optional isolated_fit/incremental_fit flags

## Outputs

- Optimal flow assignment from compound hubs to peak nodes
- Concentration factors alpha_k per compound (fraction of maximal capacity used)
- Detection indicators (assignment weight vs threshold theta) per compound
- mcf-results pickle (.pickle), results csv, and optional plotted overlay graphic (svg/png/eps/jpg)

## How to apply

Read the target spectrum and compound library as csv coordinate/weight tables (columns '1H', '13C', optionally 'weights', plus 'name' in the library) and build a flow network: a source feeds compound hub nodes with input capacity equal to each compound's maximal amount; compound-peak nodes are intermediate nodes; capacity-limited mixture nodes receive flow from every compound whose peak falls within the assignment radius r of the target peak; and an absorption sink absorbs unmatched target capacity at per-unit cost c_phi chosen larger than r. Add edges from compound-peak nodes to mixture nodes weighted by the difference between compound and target peak intensities, then impose the four feasibility constraints: flow conservation at all nodes, capacity limits on compound source flows, capacity limits on mixture nodes (target total intensity), and the joint capacity constraint preventing simultaneous over-assignment of multiple compounds onto shared target peaks. Solve the linear program with HiGHS via scipy.optimize.linprog, extract the optimal compound-hub-to-peak flows, and compute the concentration factor alpha_k for each compound as the fraction of its maximal capacity actually used; detection is indicated when the assignment weight exceeds the threshold theta. If incremental_fit is enabled, run a multipass fit over a sequence of assignment radii that gradually picks the closest target peaks and reserves their capacity; otherwise a single-pass optimization is used, and isolated_fit=false ensures assigned weights jointly respect the maximal target capacity.

## Related tools

- **mcfNMR** (Reference implementation that builds the flow network, runs the LP, and emits assignment flows, concentration factors, and detection indicators from library/target csv files) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (LP solver backend used to optimize the minimum cost flow objective (eq 8) through the SciPy interface)
- **SciPy** (Python interface through which HiGHS is invoked (scipy.optimize.linprog) for the linear optimization of flow assignment)

## Examples

```
mcfNMR -c data/user_templates/config_basic.toml
```

## Evaluation signals

- The LP must solve to optimality under HiGHS with all four feasibility constraints (flow conservation, compound capacity, mixture node capacity, joint capacity) satisfied
- Returned assignment flows must jointly respect the maximal target capacity when isolated_fit is disabled
- Concentration factors alpha_k must lie between 0 and 1 as fractions of each compound's maximal capacity
- Detections must be reproducible against a blank spectrum: the number of detections in a blank spectrum with the same noise characteristics should stay near zero at the chosen threshold theta
- Test suite (scripts/run_tests.sh) must pass on the second run due to caching operations in the first pass

## Limitations

- Overlapping peaks impede complete reconstruction of complex mixtures since peaks have considerable extension and overlaps occur frequently
- Peaks may shift, change shape, or disappear depending on sample matrix and temperature, causing sensitivity toward peak shifts
- Diminishing performance for mixtures of increasing complexity; limited quantification accuracy (at best semiquantitative, demonstrated for up to 34 compounds)
- Reliance on choice of parameters (assignment radius r, absorption cost c_phi, detection threshold theta) and method variant (single-pass vs incremental) with no definitive selection protocol
- Exact impact of the library containing more compounds than the mixture is difficult to predict for quantification

## Evidence

- [methods] Four feasibility constraints structure the LP: [evidence span withheld — anchor status: not_found]
- [intro] HiGHS via SciPy implements eq 8: "The optimization of eq 8 is implemented using HiGHS."
- [intro] Assignment radius constrains matching locally: "determining whether a peak of a spectrum X appears in Y requires a locally restricted comparison of intensities. We addressed this problem by introducing an assignment radius r"
- [abstract] mcfNMR solves EMD-style reconstruction: "a minimum cost flow problem is solved on a network consisting of nodes that represent spectral peaks of the library compounds and the mixture"
- [readme] Library/target csv formats are required: "Library- and target files must be csv-files, with columns '1H', '13C', and optionally 'weights'. A library file must additionally have a column 'name' to indicate the compound id the corresponding"
- [discussion] Single-pass vs incremental flow construction: "We explored two approaches for flow construction: a single pass MCF optimization and an incremental assignment"
