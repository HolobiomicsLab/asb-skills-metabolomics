---
name: svm-peak-quality-classification
description: Use when preprocessing large-scale GC-MS metabolomics data with unreliable
  automated peak detection, and you need to classify detected peaks as high/medium/low
  quality before downstream statistical analysis.
when_to_use_negative:
- Your peak detection is already reliable and validated — the paper targets the situation
  where 'Lack of reliable peak detection impedes automated analysis of large-scale
  GC-MS metabolomics datasets'
- Your data come from a different instrument or acquisition method than the one used
  to train the SVM classifiers — retraining on new annotations is required
- Input is already a curated feature/peak table (no raw chromatograms to classify)
edam_topics:
- http://edamontology.org/topic_3389
- http://edamontology.org/topic_3474
tools:
- name: WiPP
  role: Implements the annotation tool, SVM classifier training/hyperparameter optimisation,
    and classifier-driven peak classification
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which the WiPP preprocessing workflow and its machine learning
    classification are built
- name: conda
  role: Dependency environment manager used to install and run WiPP
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:fc1f93f772affbafccb80760e0263a818f8f26124c752fe996cd8ade0d799082
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  - conda
  - git
  - libnetcdf11
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3389
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# svm-peak-quality-classification

## Summary

Train and apply an SVM-based peak quality classifier (seven peak classes) to score detected GC-MS peaks in the WiPP workflow, so that only high-quality peaks are retained and merged across peak-picking algorithms.

## When to use

Use when preprocessing large-scale GC-MS metabolomics data with unreliable automated peak detection, and you need to classify detected peaks as high/medium/low quality before downstream statistical analysis.

## When NOT to use

- Your peak detection is already reliable and validated — the paper targets the situation where 'Lack of reliable peak detection impedes automated analysis of large-scale GC-MS metabolomics datasets'
- Your data come from a different instrument or acquisition method than the one used to train the SVM classifiers — retraining on new annotations is required
- Input is already a curated feature/peak table (no raw chromatograms to classify)

## Inputs

- GC-MS raw chromatogram files (mzML, mzData, or NetCDF) organised under an Input_folder with per-condition subdirectories plus Training_samples and Optimization_samples
- Manual peak-quality annotations from the built-in visualisation tool (seven classes, default 1200 peaks per algorithm)
- config.yaml project settings (absolute_path, high_resolution, training_data-general and optimization-general paths)

## Outputs

- Trained, reusable SVM peak-quality classifier (per peak-picking algorithm)
- Per-peak quality class assignments for the full dataset
- Merged high-quality peak table across centWave and matchedFilter for downstream analysis

## How to apply

First generate training data with run_WiPP.sh tr from a representative subset of pooled or per-condition samples (Training_samples directory), then use the built-in PDF-based visualisation tool (run_WiPP.sh an) to manually assign each candidate peak one of seven quality classes — default requirement is 1200 peaks per peak-picking algorithm (e.g. centWave and matchedFilter), done once per instrument/protocol. Launch run_WiPP.sh pp to perform SVM hyperparameter optimisation, then apply the trained classifier to every peak detected by each algorithm on the full dataset stored as mzML, mzData or NetCDF. The per-peak quality labels drive both the automated scoring of candidate detection parameters (rewarding classes A–C, penalising noise and intermediate classes) and the final result integration, in which classifier output is merged across algorithms into one high-quality peak set; medium- and low-quality peaks are kept for further inspection.

## Related tools

- **WiPP** (Implements the annotation tool, SVM classifier training/hyperparameter optimisation, and classifier-driven peak classification) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which the WiPP preprocessing workflow and its machine learning classification are built)
- **conda** (Dependency environment manager used to install and run WiPP)

## Examples

```
cd projects/example_project && ../../run_WiPP.sh tr -n 4 && ../../run_WiPP.sh an && ../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The trained classifier assigns each detected peak exactly one of the seven available peak-quality classes
- Classifier hyperparameter optimisation runs as part of the pp stage before classification of the full dataset
- Merged high-quality peak count must equal or exceed that of either single algorithm (e.g. matchedFilter 88 vs centWave 144 high-quality peaks in dataset 1, with the integrated set ≥ both)
- On the MTBLS105 case study, 100% of detected peaks should be labelled high quality and 6/9 manually confirmed analytes recovered
- A reused classifier (config.yaml `classifier: path` block) must have been trained on data from the same instrument and data acquisition method

## Limitations

- Peak annotation is labour-intensive: 1200 peaks per algorithm by default, though it only needs to be done once per instrument/protocol
- A classifier trained on too few peaks (e.g. the 25-peak example project) is likely to be inaccurate; the provided example classifier is specific to the test data and must not be used for other projects
- Ubuntu 18 (Bionic Beaver) is not yet supported due to lacking support of incorporated R packages

## Evidence

- [abstract] WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes.: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [abstract] The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set for immediate: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [readme] By default, seven different classes are available for you to choose from.: "By default, seven different classes are available for you to choose from."
- [readme] Peak annotation usually takes several hours (1200 peaks per algorithm by default), but you only need to do that once per instrument/protocol.: "Peak annotation usually takes several hours (1200 peaks per algorithm by default), but you only need to do that once per instrument/protocol. Once trained, the SVM classifiers can be reused on other"
- [readme] classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
