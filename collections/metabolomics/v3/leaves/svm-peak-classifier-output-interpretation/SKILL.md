---
name: svm-peak-classifier-output-interpretation
description: Decision support for interpreting or post-processing per-peak quality
  class assignments from WiPP's SVM classifier, e.g. when filtering detected GC-MS
  peaks by quality class or merging classified peak sets across algorithms.
when_to_use_negative:
- Peaks have not yet been annotated to train the classifier — without manual peak
  annotation (run_WiPP.sh an) there is no trained SVM model to interpret
- The task is raw peak detection or parameter optimization itself, rather than interpretation
  of already-classified peak output
- Downstream analysis requires only a pre-existing feature/peak table with no quality
  class labels — no classifier interpretation is needed
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: WiPP
  role: Provides the seven-class SVM peak quality classifier and the classification
    output that this skill interprets; run via run_WiPP.sh pp for classification and
    result integration
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which WiPP and its classifier are implemented; used for loading
    classified tables and filtering/counting high-quality peaks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:1f838ae1c6c115bc050915ee5c9eae474d46cb76c0b19e5b88f3c73d6447f40f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# SVM peak classifier output interpretation

## Summary

Interprets the output of WiPP's seven-class SVM-based peak quality classifier for GC-MS peak picking, mapping each detected peak to a quality class (high/medium/low) so that a final high-quality peak set can be assembled from multiple peak-picking algorithms (e.g., centWave and matchedFilter).

## When to use

Decision support for interpreting or post-processing per-peak quality class assignments from WiPP's SVM classifier, e.g. when filtering detected GC-MS peaks by quality class or merging classified peak sets across algorithms.

## When NOT to use

- Peaks have not yet been annotated to train the classifier — without manual peak annotation (run_WiPP.sh an) there is no trained SVM model to interpret
- The task is raw peak detection or parameter optimization itself, rather than interpretation of already-classified peak output
- Downstream analysis requires only a pre-existing feature/peak table with no quality class labels — no classifier interpretation is needed

## Inputs

- Classified peak-detection tables from WiPP's peak picking stage (one per algorithm, e.g., centWave and matchedFilter), each peak labeled with one of seven quality classes
- GC-MS raw data in mzML, mzData, or NetCDF format (indirectly, as source of the detected peaks)
- Optionally, the integrated result set from the result-integration substep

## Outputs

- A filtered high-quality peak set per algorithm (e.g., counts like 88/137 high-quality peaks for matchedFilter and 144/238 for centWave in dataset 1)
- A merged high-quality peak set across algorithms for downstream analysis
- Summary comparison tables such as peak_counts_comparison.csv (per-algorithm vs. merged high-quality peak counts)

## How to apply

WiPP evaluates every peak detected on the full dataset by an SVM-based classifier that assigns one of seven peak classes representing peak quality. Read the classifier output table produced in the 'output classification and result integration' substep of the peak-picking stage (run_WiPP.sh pp), then filter peaks to the high-quality class to form the final high-quality peak set used for downstream analysis; medium- and low-quality peaks are deliberately retained for further inspection rather than discarded. When multiple peak-picking algorithms are enabled (e.g., centWave and matchedFilter), the per-algorithm classified outputs only partially overlap, so merge the high-quality subsets of each to increase overall high-quality peak coverage. Classifiers are trained on manually annotated peaks (e.g., via run_WiPP.sh an, which presents peaks in a visualization tool for assignment into one of seven classes) and, once trained on an instrument/protocol, can be reused on other datasets generated the same way.

## Related tools

- **WiPP** (Provides the seven-class SVM peak quality classifier and the classification output that this skill interprets; run via run_WiPP.sh pp for classification and result integration) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which WiPP and its classifier are implemented; used for loading classified tables and filtering/counting high-quality peaks)

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The merged high-quality peak count must equal or exceed the count from each individual algorithm (e.g., merged ≥ max(88, 144) in the MTBLS105 dataset-1 example), since algorithm outputs only partially overlap
- Every peak in the output carries exactly one of the seven quality classes; the high-quality filter subset should be a strict subset of all detected peaks
- Counts of high-quality peaks per algorithm should match the classified per-algorithm tables (e.g., 88 of 137 for matchedFilter, 144 of 238 for centWave in dataset 1)
- The summary written to peak_counts_comparison.csv should reconcile with the per-algorithm counts and the integrated result set

## Limitations

- Classifier accuracy depends on the amount and quality of manually annotated peaks; annotating only a very small number of peaks (e.g., 25 in the example project) yields an inaccurate classifier — the provided example classifier is specific to the test data and must not be used on real projects
- Ubuntu 18 is not yet supported due to lacking support of incorporated R packages
- Peak annotation is costly: up to 1200 peaks per algorithm by default, though a trained classifier can be reused across datasets from the same instrument and acquisition method

## Evidence

- [abstract] WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes.: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [abstract] The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [abstract] Medium- and low-quality peaks are kept for further inspection.: "Medium- and low-quality peaks are kept for further inspection."
- [readme] By default, seven different classes are available for you to choose from. Once you have annotated the required amount of peaks for each algorithm (only 25 for the example project): "By default, seven different classes are available for you to choose from. Once you have annotated the required amount of peaks for each algorithm (only 25 for the example project)"
- [readme] we provide a trained classifier. In order to use this example classifier, please uncomment the line `path: example_classifier`: "we provide a trained classifier. In order to use this example classifier, please uncomment the line `path: example_classifier`"
- [readme] classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
