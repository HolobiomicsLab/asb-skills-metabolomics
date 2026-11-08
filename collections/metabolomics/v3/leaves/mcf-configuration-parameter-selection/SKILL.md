---
name: mcf-configuration-parameter-selection
description: Use when a target 2D 1H,13C HSQC spectrum (grid or peak-list csv.gz)
  must be mapped against a compound library (e.g. 501-compound MetaboMiner/Wishart
  HMDB library) with mcfNMR, and values for assignment_radius, detection_threshold,
  and fit mode must be configured.
when_to_use_negative:
- The library compound spectra are unknown or incomplete — a complete reconstruction
  requires knowledge of every constituent's peak pattern, and MCF cannot detect compounds
  absent from the library
- Precise quantification is the goal — the article lists limited quantification accuracy
  as a current limitation; results are at best semiquantitative
- The spectrum has strong temperature- or pH-induced peak shifts — the method is sensitive
  toward peak shifts, and the article offers no definitive protocol for choosing assignment_radius
  r, absorption cost cø, or detection threshold ϑ
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: mcfNMR
  role: Main solver — reads the TOML config, constructs the minimum cost flow network
    over library and target peaks, and writes mcf-results pickles and CSV tables
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: spec2csv
  role: Preprocesses .uscf, Bruker .txt.gz, or peak-list .csv inputs into the csv.gz
    target format the config's 'target' field points to, optionally with -b binning
    of grid resolution
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Linear optimizer invoked by mcfNMR to compute the optimal assignment of flow
    from source to sink nodes (accessed through SciPy)
- name: SciPy
  role: Python interface through which the HiGHS linear optimization (eq 8, minimum
    cost flow) is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:11fcb2a6a20aff382c337f2fc0ac6921388a9c1da32c161d94951e0d233631dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - SMART-Miner
  - SciPy
  - mcfNMR
  - spec2csv
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# mcf-configuration-parameter-selection

## Summary

Writes TOML configuration files for mcfNMR runs by selecting method variant (isolated_fit / incremental_fit), assignment_radius, and detection_threshold, then executing mcfNMR to produce pickled mcf-results and CSV result tables. This skill operationalizes the paper's four MCF setups (A–D) so library compounds are optimally fitted to a target spectrum via minimum cost flow.

## When to use

Use when a target 2D 1H,13C HSQC spectrum (grid or peak-list csv.gz) must be mapped against a compound library (e.g. 501-compound MetaboMiner/Wishart HMDB library) with mcfNMR, and values for assignment_radius, detection_threshold, and fit mode must be configured.

## When NOT to use

- The library compound spectra are unknown or incomplete — a complete reconstruction requires knowledge of every constituent's peak pattern, and MCF cannot detect compounds absent from the library
- Precise quantification is the goal — the article lists limited quantification accuracy as a current limitation; results are at best semiquantitative
- The spectrum has strong temperature- or pH-induced peak shifts — the method is sensitive toward peak shifts, and the article offers no definitive protocol for choosing assignment_radius r, absorption cost cø, or detection threshold ϑ

## Inputs

- Compound library CSV file with columns 'name', '1H', '13C', optionally 'weights' (e.g. 501-compound MetaboMiner/Wishart HMDB library, plain or .csv.gz)
- Target mixture spectrum file: csv/csv.gz peak table, UCSC (.uscf) grid, or Bruker matrix (.txt.gz) convertible via spec2csv; second coordinate set to a fixed value for 1D spectra
- Optional known-compound lists of the mixture (e.g. constituents of N925/N987/N988) for computing precision/recall/F1 afterwards

## Outputs

- mcf-results pickle (.pickle) containing per-compound assigned flows and containment/concentration results
- Results CSV table (.csv) of detections and assigned weights per compound
- Optional graphics (svg/png/eps/jpg) of matched compounds overlaid on the target spectrum when plot=true
- Downstream precision, recall, and F1 scores against known mixture composition (computed from the mcf-results)

## How to apply

Compose a TOML config (template data/user_templates/config_basic.toml) with obligatory fields lib, target, assignment_radius, and detection_threshold; assignment_radius is a single float for single-pass setups or a sequence of floats when incremental_fit=true. Choose the variant: setup A (isolated_fit=false, incremental_fit=false) assigns weights jointly under maximal target capacity; setup B (isolated_fit=true) fits each compound independently; setup C (incremental_fit=true with a radius sequence) gradually picks closest target peaks and reserves their capacity, giving the best F1 (~0.83); setup D applies the same logic to peak-list targets. Set detection_threshold so detections exceed it indicate containment — in a blank spectrum of matched noise characteristics the number of detections depends on this threshold, which guides its choice; if uncertain, test a range of single-pass radii first, since single-pass assignment conveys information about the optimal radius. Run `python -m mcfnmr -c <config_file>` (or `mcfNMR -c <config_file>` when pip-installed) to solve the minimum cost flow via the SciPy/HiGHS linear optimizer and emit .pickle mcf-results, .csv tables, and optional graphics into output_dir.

## Related tools

- **mcfNMR** (Main solver — reads the TOML config, constructs the minimum cost flow network over library and target peaks, and writes mcf-results pickles and CSV tables) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **spec2csv** (Preprocesses .uscf, Bruker .txt.gz, or peak-list .csv inputs into the csv.gz target format the config's 'target' field points to, optionally with -b binning of grid resolution) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Linear optimizer invoked by mcfNMR to compute the optimal assignment of flow from source to sink nodes (accessed through SciPy))
- **SciPy** (Python interface through which the HiGHS linear optimization (eq 8, minimum cost flow) is executed)

## Examples

```
python -m mcfnmr -c config_setupC.toml  # or: mcfNMR -c config_setupC.toml
```

## Evaluation signals

- On artificial mixtures N925/N987/N988 with the 501-compound library, the four setups should reach F1 ≈ 0.83 (C, incremental), ≈ 0.81 (A), ≈ 0.80 (B), ≈ 0.78 (D), exceeding MetaboMiner (0.65), COLMAR-HSQC (0.64) and SMART-Miner (0.68) by a relative margin of ≈15%
- Config parses: mcfNMR starts without complaining about missing obligatory fields (lib, target, assignment_radius, detection_threshold) and output_dir defaults to MCFNMR_HOME/output/cache when unset
- With incremental_fit=true the assignment_radius field must be a sequence of floats; with incremental_fit=false it must be a single float — a mismatch is a configuration error
- Threshold calibration sanity check: the number of detections in a blank spectrum with matched noise characteristics should depend on the chosen detection_threshold, and near-threshold detections should be traceable to that dependency
- Result files exist as expected: <name>.pickle and <name>.csv in output_dir (name auto-derived from config parameters if `name` is unset), and `load=true` reuses previously computed results

## Limitations

- No definite answer or protocol exists for choosing assignment radius r, absorption cost cø, or detection threshold ϑ; values must be tuned empirically
- Reliance on choice of parameters and method variant is an acknowledged current limitation
- Sensitivity toward peak shifts limits robustness when sample matrix or temperature moves peaks
- Performance diminishes for mixtures of increasing complexity, and quantification accuracy is limited
- The aggregate_runs test suite can take several hours on the first run; tests must be executed twice (caching) and may fail if subprocess linear-optimization output interleaves with main-process output
- Original MetaboMiner website unreachable (as of Feb 19 2025) — demodata script fetches its files from web.archive.org

## Evidence

- [readme] Config fields: obligatory lib/target/assignment_radius/detection_threshold; optional isolated_fit, incremental_fit, output_dir, plot: "The configuration file (placeholder `<config_file>` above) is a toml-file (see 'data/user\_templates/config\_basic.toml') with the following obligatory fields:"
- [readme] incremental_fit requires a sequence of assignment radii: "a sequence of assignment radii must be specified at 'assignment_radius'. Otherwise a single-pass optimization is used."
- [readme] Detection threshold semantics: assignment > threshold indicates compound containment: "Detection threshold (assignment > th indicates containment)"
- [readme] Execution command for the pip-installed entry point: "mcfNMR -c <config_file>"
- [discussion] Choice of detection threshold can be guided by detection counts in a blank spectrum of same noise: "may thus be obtained by observing the dependency of the number of detections in a blank spectrum with the same noise characteristics on the threshold value"
- [discussion] Single-pass assignment informs the optimal assignment radius: "testing single pass assignment conveys important information about the optimal assignment radius for all setups"
- [discussion] No definite protocol exists for parameter selection (r, cø, ϑ): "To date, we have no definite answer or protocol to choose these values"
- [results] Best-setup F1 0.83 vs baselines (MetaboMiner 0.65, COLMAR-HSQC 0.64, SMART-Miner 0.68): [evidence span withheld — anchor status: not_found]
- [abstract] MCF outperforms baselines by a relative margin of 15%: [evidence span withheld — anchor status: not_found]
- [discussion] Reliance on parameter/variant choice as a limitation: "(i) reliance on choice of parameters and method variant"
- [discussion] Choice of method variant is a decision point: single-pass vs incremental: [evidence span withheld — anchor status: not_found]
