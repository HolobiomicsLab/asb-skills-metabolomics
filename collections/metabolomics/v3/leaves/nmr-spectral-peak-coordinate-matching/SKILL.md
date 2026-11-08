---
name: nmr-spectral-peak-coordinate-matching
description: Use when a target NMR mixture spectrum and a library of compound spectra
  are both available as coordinate/weight tables (csv with '1H', '13C', optional 'weights'
  columns) and the goal is compound identification plus concentration estimation by
  optimal spectral fitting.
when_to_use_negative:
- The mixture contains compounds whose spectral peak pattern is not known to the library,
  since a complete reconstruction requires knowledge of every constituent's peak pattern
- Peak-picked data is already available and only a coarse classification against a
  small library is needed - the method was benchmarked against MetaboMiner, COLMAR-HSQC,
  and SMART-Miner and shines on peak-list-less grid data and simultaneous optimization
  over a 501-compound library, not simple single-compound lookups
- The target spectrum suffers severe peak shifts relative to the library spectra,
  a limitation the authors explicitly list ('sensitivity toward peak shifts')
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: mcfNMR
  role: Software implementing the minimum cost flow network construction, LP solve
    via HiGHS, and result extraction (concentration factors and detections); invoked
    via 'mcfNMR -c <config_file>' or 'python -m mcfnmr -c <config_file>'
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Linear optimization backend solving the minimum cost flow program (eq 8);
    accessed through the SciPy interface
- name: SciPy
  role: Provides the interface (scipy.optimize.linprog) through which the LP is passed
    to HiGHS
- name: spec2csv
  role: Preprocessing command converting ucsf, Bruker matrix (.txt.gz), or peak-list
    (.csv) inputs into the target csv format consumed by mcfNMR
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:5a05bb9d4939b24035422013caed4c60159f5135bc4d038511ba750c4ff6b5ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - SciPy
  - mcfNMR
  - spec2csv
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# nmr-spectral-peak-coordinate-matching

## Summary

Reconstructs an NMR mixture spectrum (e.g. 2D 1H,13C HSQC) as an optimal union of library compound spectra via a minimum cost flow on a tailored network, assigning flows between compound peaks and mixture nodes to identify and semi-quantify compounds. Solves the resulting linear program with HiGHS through the SciPy interface.

## When to use

Use when a target NMR mixture spectrum and a library of compound spectra are both available as coordinate/weight tables (csv with '1H', '13C', optional 'weights' columns) and the goal is compound identification plus concentration estimation by optimal spectral fitting.

## When NOT to use

- The mixture contains compounds whose spectral peak pattern is not known to the library, since a complete reconstruction requires knowledge of every constituent's peak pattern
- Peak-picked data is already available and only a coarse classification against a small library is needed - the method was benchmarked against MetaboMiner, COLMAR-HSQC, and SMART-Miner and shines on peak-list-less grid data and simultaneous optimization over a 501-compound library, not simple single-compound lookups
- The target spectrum suffers severe peak shifts relative to the library spectra, a limitation the authors explicitly list ('sensitivity toward peak shifts')

## Inputs

- Library csv file with columns 'name', '1H', '13C', and optionally 'weights' describing peak coordinates and intensities per compound (gzipped .csv.gz accepted)
- Target (mixture) spectrum csv file with columns '1H', '13C', and optionally 'weights' (gzipped .csv.gz accepted)
- toml configuration file specifying lib, target, assignment_radius, and detection_threshold

## Outputs

- Assignment flows from compound hubs to compound-peak nodes and mixture nodes
- Concentration factors alpha_k per compound (fraction of maximal compound capacity used)
- Detection indicators (assignment weight versus detection threshold theta)
- Result files: '.pickle' mcf-results, '.csv' results, and graphics (svg/png/eps/jpg) with matched compounds plotted on the target spectrum

## How to apply

Build the flow network: a source node feeds compound hub nodes whose input capacity equals each compound's maximal amount; compound-peak nodes connect via difference-in-intensity weighted edges to capacity-limited mixture nodes within an assignment radius r, and every mixture node links to an absorption sink with cost c_phi chosen larger than r. Impose the four feasibility constraints (flow conservation, compound source capacity, mixture node capacity equal to target total intensity, and the joint capacity constraint preventing multiple compounds over-assigning shared target peaks), then solve the linear program with scipy.optimize.linprog using the HiGHS solver. Extract optimal flows from compound hubs to their peak nodes and compute each concentration factor alpha_k as the fraction of the maximal compound capacity actually used; flag detections where the assignment weight exceeds the detection threshold theta. Optionally set isolated_fit=true to fit compounds independently, or incremental_fit=true with a sequence of assignment radii for a multipass fit that reserves target-peak capacity incrementally.

## Related tools

- **mcfNMR** (Software implementing the minimum cost flow network construction, LP solve via HiGHS, and result extraction (concentration factors and detections); invoked via 'mcfNMR -c <config_file>' or 'python -m mcfnmr -c <config_file>') — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Linear optimization backend solving the minimum cost flow program (eq 8); accessed through the SciPy interface)
- **SciPy** (Provides the interface (scipy.optimize.linprog) through which the LP is passed to HiGHS)
- **spec2csv** (Preprocessing command converting ucsf, Bruker matrix (.txt.gz), or peak-list (.csv) inputs into the target csv format consumed by mcfNMR) — https://github.com/GeoMetabolomics-ICBM/mcfNMR

## Examples

```
mcfNMR -c data/user_templates/config_basic.toml
```

## Evaluation signals

- The LP must solve to optimality under HiGHS with all four feasibility constraints satisfied
- Returned assignment flows must jointly respect the maximal target capacity when isolated_fit is disabled
- Concentration factors alpha_k must be in [0, 1], representing the fraction of each compound's maximal capacity actually used
- Detections should be flagged only where the assignment weight exceeds the detection threshold theta; calibrate theta by observing the number of detections in a blank spectrum with the same noise characteristics
- The choice of assignment radius r can be validated by testing single pass assignment, which conveys important information about the optimal radius for a given setup

## Limitations

- No definite answer or protocol exists yet for choosing the assignment radius r, absorption cost c_phi, and detection threshold theta
- Sensitivity toward peak shifts: peaks may shift, change shape, or disappear depending on the sample matrix and temperature
- Diminishing performance for mixtures of increasing complexity, and limited quantification accuracy (at best semiquantitative for mixtures up to 34 compounds)
- Overlapping peaks impede complete reconstruction of complex mixtures, since peaks have considerable extension and overlaps occur frequently
- Development under Windows is not supported, yet

## Evidence

- [intro] Network construction with assignment radius and absorption cost: "determining whether a peak of a spectrum X appears in Y requires a locally restricted comparison of intensities. We addressed this problem by introducing an assignment radius r"
- [discussion] HiGHS as the LP solver: "for our implementation, we used the SciPy25 interface to HiGHS22"
- [readme] Configuration keys and csv schema for library and target: "Library- and target files must be csv-files, with columns '1H', '13C', and optionally 'weights'. A library file must additionally have a column 'name'"
- [readme] isolated_fit and incremental_fit decision flags: "isolated_fit: Boolean flag (true or false) to indicate whether all compounds shall be fitted independently. Otherwise, assigned weights must jointly respect the maximal target capacity"
- [abstract] Simultaneous optimization without peak-picking the mixture: "our approach can be used without peak-picking the mixture spectrum and simultaneously optimizes the fit of all individual compound spectra in a given library"
- [readme] Specifying a sequence of radii for incremental fit: "assignment_radius: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true)."
