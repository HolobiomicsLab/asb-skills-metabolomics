---
name: nmr-peak-list-based-mcf-variant-execution
description: Use when the target spectrum is available as a pre-picked peak list (csv
  with 1H, 13C, optional weights) and you want compound identification that is robust
  to matrix-induced peak shifts, e.g. pH-disturbed plasma samples like N907.
when_to_use_negative:
- When the target is a raw grid spectrum and you specifically want the grid-based
  MCF variant — use spec2csv with binning (-b) and the grid pipeline instead of feeding
  a pre-picked peak list.
- 'When you need semiquantitative concentration reconstruction for complex mixtures
  (up to 34 compounds): the article reports limited quantification accuracy as a current
  limitation of MCF methods.'
- When peaks are heavily overlapping and complete compound reconstruction is required
  — overlapping peaks impede complete reconstruction, and the peak-list variant still
  depends on detectable discrete peaks.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: mcfNMR
  role: Software implementing the minimum cost flow method; the entry point `mcfNMR
    -c <config>` (or `python -m mcfnmr -c <config_file>` for the development version)
    executes the peak-list-based MCF-D classification run.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: spec2csv
  role: Utility from the mcfnmr package that converts supported input formats (USCF,
    Bruker matrices txt.gz, or peak-list csv) into the target csv consumed by the
    classification run.
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: HiGHS
  role: Linear optimization backend (accessed through SciPy) that solves the minimum
    cost flow on the network of library and target peaks.
- name: SciPy
  role: Provides the Python interface through which the HiGHS optimizer is invoked
    for the flow computation.
- name: MetaboMiner
  role: Baseline algorithm whose compound library (HMDB-based, provided by the Wishart
    lab) is reused, and whose published F1 scores (76.2 on N907, 82.9 on N926) serve
    as the comparison target.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:d3979f0b48fc0ddeee466945b427adf43f41cdfbdfeb4b19f1ad2293cfceba4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - HiGHS
  - MetaboMiner
  - SciPy
  - mcfNMR
  - spec2csv
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nmr-peak-list-based-mcf-variant-execution

## Summary

Executes the peak-list-based MCF variant (MCF-D) of mcfNMR, in which library spectra are compared to pre-picked target peaks rather than a serialized grid spectrum, and evaluates detections via F1 scores against the MetaboMiner baseline.

## When to use

Use when the target spectrum is available as a pre-picked peak list (csv with 1H, 13C, optional weights) and you want compound identification that is robust to matrix-induced peak shifts, e.g. pH-disturbed plasma samples like N907.

## When NOT to use

- When the target is a raw grid spectrum and you specifically want the grid-based MCF variant — use spec2csv with binning (-b) and the grid pipeline instead of feeding a pre-picked peak list.
- When you need semiquantitative concentration reconstruction for complex mixtures (up to 34 compounds): the article reports limited quantification accuracy as a current limitation of MCF methods.
- When peaks are heavily overlapping and complete compound reconstruction is required — overlapping peaks impede complete reconstruction, and the peak-list variant still depends on detectable discrete peaks.

## Inputs

- Target peak list csv file with columns '1H', '13C', and optionally 'weights' (converted via spec2csv; .csv or .csv.gz)
- Library csv file (e.g. HMDB-based MetaboMiner plasma common library, 501 compounds) with columns 'name', '1H', '13C', and optionally 'weights'
- mcfNMR toml configuration file with lib, target, assignment_radius (single float), and detection_threshold fields; isolated_fit and incremental_fit set to false for the single-pass variant

## Outputs

- mcf-results pickle file containing per-compound assignment/detection scores
- results csv file listing compound names with detection scores above the threshold
- Optional graphic (svg/png per gfxformat) of matched compounds plotted on the target spectrum when plot=true
- F1 detection score for the target spectrum (e.g. 81.6 for N907 peak-list variant MCF-D)

## How to apply

First convert the peak list to an mcfNMR target csv using spec2csv (input with columns '1H', '13C', optionally 'weights'; the script's heuristics match column names if not present). Write a toml configuration with obligatory fields lib (path to the HMDB-based MetaboMiner library csv, 501 compounds with 'name', '1H', '13C', optionally 'weights'), target, assignment_radius (single float for the single-pass variant), and detection_threshold (calibrated by observing detections in a blank spectrum with similar noise). Set isolated_fit and incremental_fit to false to use the single-pass MCF optimization, then run the classification via the mcfNMR entry point; the underlying linear optimization (HiGHS via SciPy) computes an optimal flow from source to sink nodes whose assignment scores above the detection threshold indicate compound containment. Because the peak-list variant feeds discrete peaks rather than a binned grid (as created with spec2csv -b), it avoids peak-shape dilution and is the recommended variant when matrix conditions (e.g. pH shifts in sample N907) displace peaks. Extract the F1 detection score from the mcf-results (pickle/csv) and compare against MetaboMiner baseline values (e.g. N907: 83.5 MCF vs 76.2 MetaboMiner; N926: 81.6 vs 82.9).

## Related tools

- **mcfNMR** (Software implementing the minimum cost flow method; the entry point `mcfNMR -c <config>` (or `python -m mcfnmr -c <config_file>` for the development version) executes the peak-list-based MCF-D classification run.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **spec2csv** (Utility from the mcfnmr package that converts supported input formats (USCF, Bruker matrices txt.gz, or peak-list csv) into the target csv consumed by the classification run.) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **HiGHS** (Linear optimization backend (accessed through SciPy) that solves the minimum cost flow on the network of library and target peaks.)
- **SciPy** (Provides the Python interface through which the HiGHS optimizer is invoked for the flow computation.)
- **MetaboMiner** (Baseline algorithm whose compound library (HMDB-based, provided by the Wishart lab) is reused, and whose published F1 scores (76.2 on N907, 82.9 on N926) serve as the comparison target.)

## Examples

```
spec2csv -i N907_peaklist.csv -o target.csv.gz && mcfNMR -c config_N907.toml
```

## Evaluation signals

- Reproduced F1 scores match published values within rounding tolerance: MCF-D on N907 = 81.6 (vs MetaboMiner 76.2); MCF grid on N907 = 83.5; MCF on N926 = 81.6 (vs MetaboMiner 82.9).
- The mcf-results pickle and results csv are produced in output_dir, and detection counts remain stable when the run is repeated with load=true (cached result reuse).
- Detection threshold calibration: the number of detections in a blank spectrum with comparable noise should stay low at the chosen threshold — if blank detections rise, the threshold is too permissive.
- The assignment radius must be a single float when incremental_fit=false; if a sequence is supplied without enabling incremental_fit, the configuration is inconsistent and the run should be rejected.
- The compound names in the results csv should all belong to the 501-compound HMDB-based library used in the configuration's lib file.

## Limitations

- Performance depends on choice of parameters and method variant (assignment radius r, absorption cost, detection threshold ϑ); the authors state there is no definite answer or protocol to choose these values.
- The method is sensitive toward peak shifts; the peak-list variant mitigates but does not eliminate this, as shown by improved performance on the pH-disturbed plasma sample N907.
- Performance diminishes for mixtures of increasing complexity, and quantification accuracy is limited.
- The original MetaboMiner website cannot be reached (as of Feb 19 2025); its data must be fetched from web.archive.org via scripts/download_demodata.sh 'metabominer'.
- Development under Windows is not supported.

## Evidence

- [readme] lib, target, assignment_radius and detection_threshold are obligatory config fields; the single-pass variant sets isolated_fit and incremental_fit to false: "with the following obligatory fields:

- `lib`: Path to a library-file containing the names, peak-weights, and -coords"
- [readme] spec2csv supports peak-list csv inputs with columns 1H, 13C and optionally weights: "and peak lists (.csv) with columns '1H', '13C', and optionally 'weights' (some basic heuristics try to match these columns to others if not present)."
- [discussion] The article reports that the peak-list-based method improved performance on the pH-disturbed plasma sample N907: "the improved performance of the peak-list-based method (MCF-D) on the plasma sample, especially for the pH-disturbed sample N907"
- [discussion] The minimum cost flow is solved by a linear optimization (HiGHS via SciPy): "lies a linear optimization that computes an optimal assignment of
flow from source to sink nodes"
- [discussion] Detection threshold is calibrated by observing detections in a blank spectrum: "may thus be obtained by observing the dependency of the number of
detections in a blank spectrum with the same noise characteristics
on the threshold value"
