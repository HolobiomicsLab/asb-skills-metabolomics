---
name: peak-picking-parameter-optimization
description: Use when preprocessing large-scale GC-MS metabolomics raw data (mzML,
  mzData or NetCDF) where reliable peak detection is the bottleneck and per-algorithm
  peak-picking parameters need automated optimisation rather than manual tuning.
when_to_use_negative:
- Input is already a processed feature/peak table — WiPP works on raw GC-MS chromatogram
  files
- Data are not GC-MS metabolomics chromatograms (e.g. LC-MS or proteomics), since
  the seven peak classes and embedded peak-picking algorithms are GC-MS specific
- Peak annotation cannot be performed for the instrument/protocol and no compatible
  pre-trained classifier exists — an untrained classifier is likely to be inaccurate
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: WiPP
  role: 'Implements the full workflow: training-data generation, peak annotation,
    SVM classifier training/hyperparameter optimisation, per-algorithm parameter optimisation,
    peak detection, classification and result integration'
  repo: https://github.com/bihealth/WiPP
- name: conda
  role: Runtime environment manager; run_WiPP.sh creates the conda environment on
    first run
- name: libnetcdf11
  role: System library dependency for reading NetCDF-format GC-MS data
- name: git
  role: Used to clone the WiPP repository for installation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:39fb703ca63dba8a2c20117ef3a2d684a9cad3c4a8ade46fa6c7ee807530bc63
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# peak-picking-parameter-optimization

## Summary

Automated, machine-learning-driven optimisation of GC-MS peak-picking algorithm parameters and merging of multiple algorithms' peak lists into a single high-quality peak set, as implemented in the WiPP workflow.

## When to use

Use when preprocessing large-scale GC-MS metabolomics raw data (mzML, mzData or NetCDF) where reliable peak detection is the bottleneck and per-algorithm peak-picking parameters need automated optimisation rather than manual tuning.

## When NOT to use

- Input is already a processed feature/peak table — WiPP works on raw GC-MS chromatogram files
- Data are not GC-MS metabolomics chromatograms (e.g. LC-MS or proteomics), since the seven peak classes and embedded peak-picking algorithms are GC-MS specific
- Peak annotation cannot be performed for the instrument/protocol and no compatible pre-trained classifier exists — an untrained classifier is likely to be inaccurate

## Inputs

- GC-MS raw chromatogram files in mzML, mzData or NetCDF format
- Input_folder structured with condition subdirectories, Training_samples, Optimization_samples, and optional Wash (alkane blank) directories
- config.yaml with absolute_path, high_resolution flag, retention-index alkane list, and Training/Optimization sample paths
- Manually annotated peaks per algorithm (default 1200 per algorithm) across seven peak classes

## Outputs

- Optimised peak-picking parameters for each embedded peak detection algorithm
- Per-peak quality classification (high/medium/low) from the trained SVM classifier
- One merged final high-quality peak set for downstream analysis, with medium/low-quality peaks retained for inspection

## How to apply

Structure the Input_folder with per-condition subdirectories plus Training_samples, Optimization_samples and optional Wash (alkane blank) directories, then fill the compulsory config.yaml parameters: absolute_path, high_resolution (False for SIM-mode GC-qMS with mass resolution below 1 Da), retention-index alkane list (e.g. c10,...,c34), and relative Training/Optimization sample paths. Run `run_WiPP.sh tr` to generate training data, then annotate ~1200 peaks per algorithm with the built-in visualisation tool using the seven available peak classes (done only once per instrument/protocol; trained SVM classifiers can be reused). Launch `run_WiPP.sh pp`, which performs SVM classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration. The classifier quality labels are merged across the embedded peak-picking algorithms into one final high-quality peak table; medium- and low-quality peaks are kept for further inspection.

## Related tools

- **WiPP** (Implements the full workflow: training-data generation, peak annotation, SVM classifier training/hyperparameter optimisation, per-algorithm parameter optimisation, peak detection, classification and result integration) — https://github.com/bihealth/WiPP
- **conda** (Runtime environment manager; run_WiPP.sh creates the conda environment on first run)
- **libnetcdf11** (System library dependency for reading NetCDF-format GC-MS data)
- **git** (Used to clone the WiPP repository for installation)

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- All detected peaks receive a quality label from the classifier, and the final merged table flags 100% of peaks as high quality on a well-trained instrument/protocol (as in the MTBLS105 case study)
- Optimisation uses different samples than classifier training — Optimization_samples must not overlap Training_samples
- Confirmed analytes recovered in the merged high-quality peak table show fold changes and p-values comparable to a reference manual analysis (6 of 9 analytes auto-identified on MTBLS105, 3 flagged as shouldering)
- Pipeline completes all substeps in the 'pp' stage: classifier hyperparameter optimisation, algorithm parameter optimisation, full-dataset peak detection, output classification and result integration
- Number of annotated peaks per algorithm reaches the default requirement (1200) before the peak-picking stage

## Limitations

- Peak annotation takes several hours (1200 peaks per algorithm by default), though it is only needed once per instrument/protocol
- Using too few annotated peaks makes the classifier likely inaccurate; the bundled example classifier is specific to the test data and must not be used for real projects
- Ubuntu 18 (Bionic Beaver) is not supported due to lacking support of incorporated R packages; tested on Ubuntu 16 and CentOS 7.6.1810

## Evidence

- [abstract] WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes.: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [abstract] Final peak set built by merging classifier output across algorithms.: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [readme] Installation and execution steps from README.: "git clone https://github.com/bihealth/WiPP.git
cd WiPP
make"
- [readme] Substeps of the peak-picking stage.: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
- [readme] Reuse of trained classifiers across datasets from the same instrument.: "Once trained, the SVM classifiers can be reused on other datasets generated with the same instrument and data acquisition method."
- [readme] Supported input file formats.: "**WiPP** supports mzML, mzData and NetCDF file formats."
