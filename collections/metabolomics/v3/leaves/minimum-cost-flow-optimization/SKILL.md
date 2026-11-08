---
name: minimum-cost-flow-optimization
description: Use when you have 2D 1H,13C HSQC NMR spectra of a mixture (grid data
  or peak lists) and a compound library, and need to identify/quantify constituents
  without peak-picking the mixture spectrum.
when_to_use_negative:
- Mixture constituents whose peak pattern is unknown, so no library spectra exist
  to fit (the article notes 'the mixture may contain compounds whose spectral peak
  pattern is not known').
- Samples where a peak-picking preprocessing step is already required by the downstream
  pipeline (the MCF approach can 'be used without peak-picking the mixture spectrum').
- Extremely complex mixtures with severe peak overlap where the article reports 'diminishing
  performance for mixtures of increasing complexity' and 'limited quantification accuracy'.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: mcfNMR
  role: Reference implementation that reads library/target csv files and a TOML config,
    constructs the flow network, and solves the minimum cost flow problem to recover
    constituent compounds.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Backend linear optimizer invoked (via SciPy) to compute the optimal assignment
    of flow from source to sink nodes in the flow network.
- name: SciPy
  role: Python interface through which HiGHS is called for the linear optimization
    underlying the MCF solution.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:78c3fec80198c0010375d0b78c582c5f2544b0724fd733def16af4de1747eb89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - SMART-Miner
  - SciPy
  - mcfNMR
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# minimum-cost-flow-optimization

## Summary

Fit compound spectra to an NMR mixture spectrum by casting peak-to-peak intensity matching as a minimum cost flow problem on a network whose nodes represent spectral peaks of library compounds and the mixture, solved with a linear optimizer.

## When to use

Use when you have 2D 1H,13C HSQC NMR spectra of a mixture (grid data or peak lists) and a compound library, and need to identify/quantify constituents without peak-picking the mixture spectrum.

## When NOT to use

- Mixture constituents whose peak pattern is unknown, so no library spectra exist to fit (the article notes 'the mixture may contain compounds whose spectral peak pattern is not known').
- Samples where a peak-picking preprocessing step is already required by the downstream pipeline (the MCF approach can 'be used without peak-picking the mixture spectrum').
- Extremely complex mixtures with severe peak overlap where the article reports 'diminishing performance for mixtures of increasing complexity' and 'limited quantification accuracy'.

## Inputs

- Library csv file with columns 'name', '1H', '13C', and optionally 'weights' (e.g. 501-compound MetaboMiner/Wishart HMDB-based library)
- Target csv/csv.gz file with columns '1H', '13C', and optionally 'weights' (converted from UCSC .uscf, Bruker .txt.gz grid spectra, or peak lists via spec2csv)
- TOML configuration file specifying lib, target, assignment_radius, detection_threshold, isolated_fit, incremental_fit

## Outputs

- Pickled mcf-results (.pickle) containing per-compound assignments and assigned weights/concentrations
- CSV result table listing detected compounds and their assigned weights
- Optional plots of matched compounds overlaid on the target spectrum (svg/png/eps/jpg depending on gfxformat)

## How to apply

Represent the mixture target spectrum (converted to a csv/csv.gz file with '1H', '13C', 'weights' columns via spec2csv) and library compound spectra (a csv file with an added 'name' column) as nodes of a bipartite network, then solve a minimum cost flow problem where each source peak of a compound must send a unit of flow to the target sink whose coordinate lies within the assignment radius r. Set the assignment_radius as a single float for a single-pass MCF optimization or a sequence of floats for an incremental multipass fit (isolated_fit=false, incremental_fit=true), or isolated_fit=true to fit each compound independently while respecting a maximal target capacity jointly. Choose detection_threshold so that an assignment value above it indicates compound containment; tune absorption cost cø and detection threshold by observing detections in a blank spectrum with the same noise characteristics. Execute via a TOML config file consumed by python -m mcfnmr, which invokes the linear optimizer (SciPy interface to HiGHS) and writes pickled mcf-results, CSV result tables, and optional plots showing matched compounds overlaid on the target spectrum.

## Related tools

- **mcfNMR** (Reference implementation that reads library/target csv files and a TOML config, constructs the flow network, and solves the minimum cost flow problem to recover constituent compounds.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Backend linear optimizer invoked (via SciPy) to compute the optimal assignment of flow from source to sink nodes in the flow network.)
- **SciPy** (Python interface through which HiGHS is called for the linear optimization underlying the MCF solution.)

## Examples

```
python -m mcfnmr -c config_mixtures.toml
```

## Evaluation signals

- F1 score against known constituent lists of artificial mixtures (e.g. N925/N987/N988) should match reported values: setup C ~0.83, A ~0.81, B ~0.80, D ~0.78, exceeding MetaboMiner (0.65), COLMAR-HSQC (0.64) and SMART-Miner (0.68) by ~15% relative margin.
- The mcf-results pickle must contain, for every library compound, either no assignment or an assignment value exceeding detection_threshold to indicate containment.
- The assigned weights must jointly respect the maximal target capacity unless isolated_fit=true, and the solution must be a valid flow conservation solution produced by the HiGHS/SciPy optimizer.
- Re-running the same configuration must reproduce identical mcf-results (mcfNMR caches results and warns if 'load' is used to reuse an existing pickled result).
- If plot=true, the generated graphic should show each detected compound's peaks overlaid on the target spectrum within the specified assignment_radius.

## Limitations

- Reliance on choice of parameters and method variant: no definite answer or protocol exists to choose assignment radius r, absorption cost cø, and detection threshold ϑ.
- Sensitivity toward peak shifts (peaks may shift, change shape, or disappear depending on sample matrix and temperature).
- Diminishing performance for mixtures of increasing complexity and limited quantification accuracy.
- Overlapping peaks impede complete reconstruction; the method cannot recover compounds absent from the library.

## Evidence

- [abstract] Nodes represent spectral peaks of library compounds and the mixture: "a minimum cost flow problem is solved on a network consisting of nodes that represent spectral peaks of the library compounds and the mixture"
- [intro] Assignment radius restricts peak matching locally: "determining whether a peak of a spectrum X appears in Y requires a locally restricted comparison of intensities. We addressed this problem by introducing an assignment radius r"
- [intro] Optimization implemented using HiGHS: "The optimization of eq 8 is implemented using HiGHS."
- [discussion] SciPy interface to HiGHS: "for our implementation, we used the SciPy25 interface to HiGHS22"
- [readme] Config file fields and boolean flags: "- `isolated_fit`: Boolean flag (true or false) to indicate whether all compounds shall be fitted independently. Otherwise, assigned weights must jointly respect the maximal target capacity. (default:"
- [readme] Incremental fit requires sequence of radii: "- `incremental_fit`: Boolean flag (true or false) to indicate whether a multipass incremental fit should be performed. In this case, a sequence of assignment radii must be specified at"
- [readme] Detection threshold indicates containment: "- `detection_threshold`:  Detection threshold (assignment > th indicates containment)."
- [readme] Library file format with name column: "Library- and target files must be csv-files, with columns '1H', '13C', and optionally 'weights'. A library file must additionally have a column 'name'"
- [discussion] No protocol to choose parameters: "To date, we have no definite answer or protocol to choose these values"
- [discussion] Method outperforms other algorithms: "our method achieved higher F1 scores than all other methods (by a relative margin of 15%)"
