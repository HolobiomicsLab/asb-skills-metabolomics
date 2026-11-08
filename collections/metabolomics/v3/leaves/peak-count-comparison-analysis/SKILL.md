---
name: peak-count-comparison-analysis
description: Use when GC-MS peak picking has been run with multiple embedded algorithms
  (e.g. centWave and matchedFilter) via WiPP and you need to quantify whether the
  integrated result set yields larger high-quality peak coverage than either algorithm
  alone.
when_to_use_negative:
- Only a single peak-picking algorithm was enabled in the WiPP config.yaml (e.g. the
  example project defaults to one algorithm), so there is no second algorithm's output
  to compare against.
- Input is a raw GC-MS data file (mzML, mzData, NetCDF) that has not yet been through
  WiPP's peak picking, classification, and result-integration stages.
- The goal is parameter optimisation or classifier training rather than evaluating
  high-quality peak coverage of the integrated result set.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: WiPP
  role: Executes peak picking with multiple embedded algorithms, SVM-based seven-class
    peak quality classification, and result integration whose high-quality peak counts
    are compared
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which WiPP is built and in which the comparison/counting script
    is implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:8fee743b39063c224a8432406a6bac155664c7e4b59d3b74c026f378289417b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# peak-count-comparison-analysis

## Summary

Compares the number of high-quality peaks (as assigned by WiPP's seven-class SVM peak quality classifier) produced by the merged/integrated result set versus each individual peak-picking algorithm (e.g. centWave and matchedFilter), to quantify the coverage gain from result integration across algorithms.

## When to use

Use when GC-MS peak picking has been run with multiple embedded algorithms (e.g. centWave and matchedFilter) via WiPP and you need to quantify whether the integrated result set yields larger high-quality peak coverage than either algorithm alone.

## When NOT to use

- Only a single peak-picking algorithm was enabled in the WiPP config.yaml (e.g. the example project defaults to one algorithm), so there is no second algorithm's output to compare against.
- Input is a raw GC-MS data file (mzML, mzData, NetCDF) that has not yet been through WiPP's peak picking, classification, and result-integration stages.
- The goal is parameter optimisation or classifier training rather than evaluating high-quality peak coverage of the integrated result set.

## Inputs

- Per-algorithm classified peak-detection tables (e.g. centWave and matchedFilter) with seven-class peak quality labels, from the WiPP peak picking stage on the full dataset
- Integrated result set from WiPP's result-integration substep

## Outputs

- High-quality peak counts per algorithm and for the merged set
- Coverage-difference summary table (peak_counts_comparison.csv)

## How to apply

Load the per-algorithm classified peak-detection tables produced by the WiPP peak picking stage (run_WiPP.sh pp) on the full dataset, where each peak carries one of the seven SVM-based peak quality classes. Filter each table to peaks assigned the high-quality class, and count high-quality peaks separately for each algorithm (e.g. centWave and matchedFilter). Load the integrated result set from WiPP's result-integration substep — which merges classifier quality information with results from different peak detection algorithms into one final high-quality peak set — and count its high-quality peaks. Compute the coverage difference of the merged set versus each individual algorithm and write the comparison to a summary table such as peak_counts_comparison.csv. The rationale is that the algorithms' outputs only partially overlap, so integration across embedded peak picking algorithms increases high-quality peak coverage; in the MTBLS105 case study, matchedFilter found 137 peaks (88 high-quality) and centWave 238 peaks (144 high-quality), while the merged set exceeded both.

## Related tools

- **WiPP** (Executes peak picking with multiple embedded algorithms, SVM-based seven-class peak quality classification, and result integration whose high-quality peak counts are compared) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which WiPP is built and in which the comparison/counting script is implemented)

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The merged high-quality peak count should equal or exceed the count from each individual algorithm (integration across algorithms improves coverage).
- Per-algorithm counts should be reproducible from the classified peak tables after filtering to the high-quality class (e.g. 88 high-quality of 137 matchedFilter peaks; 144 of 238 centWave peaks in the MTBLS105 dataset 1).
- The comparison output table (peak_counts_comparison.csv) is written and contains one coverage-difference value per algorithm versus the merged set.
- The high-quality class labels come from the SVM classifier's seven-class scheme, not manual filtering, so label provenance can be checked against the classifier output.
- Counts should reflect peaks from the full-dataset detection stage, not the training or optimization sample subsets.

## Limitations

- Partial overlap between algorithms means merged counts can exceed individual counts, but the size of the gain is dataset-dependent and cannot be assumed a priori.
- The SVM classifier quality depends on the amount and representativeness of manually annotated training peaks (default 1200 peaks per algorithm; example project uses only 25); WiPP provides a reusable trained classifier only for the same instrument/data acquisition method.
- Ubuntu 18 (Bionic Beaver) is not supported due to lacking support of incorporated R packages, which may block reproducing the pipeline on some systems.

## Evidence

- [full_text] Merging the two algorithms increases the number of peaks classified as high-quality, since their outputs only partially overlap; e.g., in dataset 1 matchedFilter found 137 peaks (88 high-quality) and centWave 238 peaks (144 high-quality).: [evidence span withheld — anchor status: not_found]
- [abstract] The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set for immediate: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [readme] Many sequential substeps are in fact happening during this final step of the pipeline such as classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration.: "peak detection on the full dataset, output classification and result integration"
- [readme] By default, in the example project, we run only one peak picking algorithm to speed up the test. Do not forget to enable in the config.yaml file the peak picking algorithms you want to use.: "we run only one peak picking algorithm to speed up the test. Do not forget to enable in the `config.yaml` file the peak picking algorithms you want to use"
