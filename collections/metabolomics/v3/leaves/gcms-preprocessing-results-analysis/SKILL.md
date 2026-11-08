---
name: gcms-preprocessing-results-analysis
description: Use when you have per-algorithm classified peak-detection tables from
  a WiPP run and need to measure whether the merged/integrated high-quality peak set
  exceeds each individual algorithm's high-quality yield.
when_to_use_negative:
- Input is a single-algorithm peak table with no classifier quality labels — the comparison
  requires high-quality-class assignments per algorithm plus the integrated set
- Data was not preprocessed with WiPP's multi-algorithm peak picking (e.g., an LC-MS
  feature table or an already-final peak list); this skill analyzes result integration,
  not raw peak picking
- The goal is training or annotating the SVM classifier (run_WiPP.sh tr / an stages)
  rather than evaluating the integrated high-quality peak set
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: WiPP
  role: Produces the per-algorithm classified peak tables and the integrated high-quality
    peak set that this analysis compares
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Runtime in which the peak-count comparison and summary-table export are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:0485e70051aaaddaeb500c99e5834c60c1ac8013189a24d705e0b358cc85e95a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gcms-preprocessing-results-analysis

## Summary

Quantifies how much high-quality peak coverage improves when WiPP integrates results across embedded peak-picking algorithms (centWave and matchedFilter) versus using either algorithm alone, by comparing counts of peaks assigned the high-quality class by the seven-class SVM-based classifier.

## When to use

Use when you have per-algorithm classified peak-detection tables from a WiPP run and need to measure whether the merged/integrated high-quality peak set exceeds each individual algorithm's high-quality yield.

## When NOT to use

- Input is a single-algorithm peak table with no classifier quality labels — the comparison requires high-quality-class assignments per algorithm plus the integrated set
- Data was not preprocessed with WiPP's multi-algorithm peak picking (e.g., an LC-MS feature table or an already-final peak list); this skill analyzes result integration, not raw peak picking
- The goal is training or annotating the SVM classifier (run_WiPP.sh tr / an stages) rather than evaluating the integrated high-quality peak set

## Inputs

- Per-algorithm classified peak-detection tables from the WiPP peak picking (pp) stage on the full dataset (one for centWave, one for matchedFilter), each with a seven-class quality label per peak
- Integrated result set from WiPP's result-integration substep
- Python 3 environment

## Outputs

- peak_counts_comparison.csv: high-quality peak counts per algorithm and for the merged set, with coverage differences
- Summary counts of high-quality peaks for centWave, matchedFilter, and the integrated set

## How to apply

Load the per-algorithm classified peak-detection tables (centWave and matchedFilter) emitted by the WiPP peak picking stage on the full dataset. Filter each table to peaks assigned the high-quality class by the seven-class SVM-based peak quality classifier, and count high-quality peaks per algorithm. Load the integrated result set from WiPP's result-integration substep and count its high-quality peaks. Compute the coverage difference of the merged set versus each individual algorithm (e.g., in MTBLS105 dataset 1, matchedFilter found 137 peaks with 88 high-quality and centWave 238 peaks with 144 high-quality, so the merged set must reach or exceed 144) and write the summary to peak_counts_comparison.csv. The rationale is that the two algorithms detect partially non-overlapping peaks, so result integration across peak picking algorithms improves high-quality peak coverage.

## Related tools

- **WiPP** (Produces the per-algorithm classified peak tables and the integrated high-quality peak set that this analysis compares) — https://github.com/bihealth/WiPP
- **Python 3** (Runtime in which the peak-count comparison and summary-table export are implemented)

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The merged high-quality peak count equals or exceeds the count from each individual algorithm (invariant of result integration).
- High-quality counts are strictly lower than raw detected peak counts per algorithm (e.g., matchedFilter 88 high-quality out of 137 detected; centWave 144 out of 238 in MTBLS105 dataset 1).
- peak_counts_comparison.csv is written and contains one row/entry per algorithm plus the merged set with a coverage-difference column.
- The integrated set derives from WiPP's result-integration substep of the pp stage, not from re-running peak picking.

## Limitations

- Coverage comparison only covers algorithms embedded in WiPP (centWave and matchedFilter); other peak pickers are not evaluated.
- The seven-class SVM classifier's accuracy depends on annotated training peaks; a classifier trained on very few peaks (e.g., 25 in the example project) is likely inaccurate, and classifiers are specific to instrument/data acquisition method.
- Ubuntu 18 is unsupported due to lacking support of incorporated R packages; WiPP is tested on Ubuntu 16 and CentOS 7.6.

## Evidence

- [abstract] The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [abstract] WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes.: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [readme] peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration: "peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
- [abstract] we demonstrate that peak detection is improved through the novel way to assign peak quality, an automated parameter optimisation, and results in integration across different embedded peak picking: "we demonstrate that peak detection is improved through the novel way to assign peak quality, an automated parameter optimisation, and results in integration across different embedded peak picking"
