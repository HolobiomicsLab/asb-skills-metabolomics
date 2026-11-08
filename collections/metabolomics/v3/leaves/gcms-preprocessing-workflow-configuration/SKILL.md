---
name: gcms-preprocessing-workflow-configuration
description: Use when preprocessing large-scale GC-MS metabolomics datasets (mzML/mzData/NetCDF)
  where peak detection parameters for embedded algorithms (centWave, matchedFilter)
  must be optimised without manual tuning.
when_to_use_negative:
- When input is already a peak/feature table rather than raw GC-MS chromatogram files
  — the skill operates on raw mzML/mzData/NetCDF data
- When the only samples available are those already used for classifier training —
  parameters must be optimised on different Optimization_samples
- When a single, manually tuned peak-picking parameter set is already validated for
  the instrument/protocol and no algorithm comparison is needed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: WiPP
  role: 'Implements the full pipeline: classifier training, grid-search parameter
    optimisation via the seven-class SVM peak-quality scheme, and result integration
    across centWave and matchedFilter'
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which the WiPP preprocessing workflow is built and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:691f7d3d0db17adfe5522636761e59fa37ae79d55436744973146a4b6aedfcbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gcms-preprocessing-workflow-configuration

## Summary

Configures and runs the WiPP GC-MS preprocessing pipeline, which performs exhaustive grid-search optimisation of centWave and matchedFilter peak-picking parameters using a trained SVM peak-quality classifier and a class-distribution scoring function, then applies the optimal parameters to the full dataset.

## When to use

Use when preprocessing large-scale GC-MS metabolomics datasets (mzML/mzData/NetCDF) where peak detection parameters for embedded algorithms (centWave, matchedFilter) must be optimised without manual tuning.

## When NOT to use

- When input is already a peak/feature table rather than raw GC-MS chromatogram files — the skill operates on raw mzML/mzData/NetCDF data
- When the only samples available are those already used for classifier training — parameters must be optimised on different Optimization_samples
- When a single, manually tuned peak-picking parameter set is already validated for the instrument/protocol and no algorithm comparison is needed

## Inputs

- GC-MS raw files in mzML, mzData or NetCDF format
- Project config.yaml (absolute_path, high_resolution, retention_index, training_data-general, optimization-general settings)
- Training_samples directory (subset of pooled or representative condition samples, >=2 per condition if no pooled samples)
- Optimization_samples directory (different samples from those used for classifier training)
- Trained SVM classifier (or provided example_classifier for testing only)

## Outputs

- optimized_parameters.yaml with the argmax-scored centWave and matchedFilter parameter sets
- Scored per-peak class distributions for each candidate parameter set
- Final merged high-quality peak set for downstream GC-MS analysis

## How to apply

Create a project directory containing config.yaml (absolute_path to Input_folder, high_resolution flag, training_data-general and optimization-general block paths) and place mzML, mzData or NetCDF files under condition subdirectories plus Training_samples and Optimization_samples subdirectories, ensuring the optimisation samples are different from those used for classifier training. Train an algorithm-specific SVM classifier that assigns peaks to one of seven peak-quality classes via the run_WiPP.sh tr (training) and an (annotation) steps. During the pp step, run centWave and matchedFilter over every candidate parameter combination on the Optimization_samples, classify each detected peak with the SVM classifier, and apply the scoring function that rewards classes A–C while penalising noise and intermediate classes to aggregate a score per parameter set. Select, per algorithm, the parameter set with the maximum score (the argmax of the classifier-driven quality score) and write the optimal centWave and matchedFilter settings to optimized_parameters.yaml. Finally, reuse the retained scored peak classifications to run peak detection on the full dataset and integrate results across the different embedded peak-picking algorithms into one final high-quality peak set.

## Related tools

- **WiPP** (Implements the full pipeline: classifier training, grid-search parameter optimisation via the seven-class SVM peak-quality scheme, and result integration across centWave and matchedFilter) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which the WiPP preprocessing workflow is built and executed)

## Examples

```
cd ./projects/example_project && ../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- optimized_parameters.yaml is produced and its centWave/matchedFilter settings equal the parameter set with the maximum classifier-driven quality score across the exhaustive grid
- Every detected peak is assigned one of the seven peak-quality classes by the algorithm-specific SVM classifier before scoring
- The scoring function rewards peaks in high-quality classes A–C and penalises noise and intermediate classes; verify the selected set has the best aggregate class distribution
- The Optimization_samples directory contains samples distinct from the Training_samples used to train the classifier
- The pipeline run from the project directory (the one containing config.yaml) completes tr, an, and pp substeps without error

## Limitations

- The provided example classifier is specific to the test data and must not be used for user projects
- Ubuntu 18 (Bionic Beaver) is not supported due to lacking support of incorporated R packages; tested on Ubuntu 16 and CentOS 7.6
- Peak annotation (~1200 peaks per algorithm by default) is time-consuming, though it only needs to be done once per instrument/protocol and SVM classifiers can be reused on datasets generated with the same instrument and acquisition method

## Evidence

- [abstract] Seven peak classes drive scoring: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [readme] Optimization samples must differ from training samples: "make sure to choose different samples as we do not want the parameters to be optimized using the same data the classifier was trained on."
- [readme] pp substep performs parameter optimisation: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
- [abstract] Peak detection improved via automated parameter optimisation: "peak detection is improved through the novel way to assign peak quality, an automated parameter optimisation, and results in integration across different embedded peak picking"
