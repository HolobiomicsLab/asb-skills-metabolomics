---
name: gcms-peak-detection-parameter-optimization
description: Use when preprocessing GC-MS metabolomics data (mzML/mzData/NetCDF) with
  WiPP and needing to select centWave or matchedFilter peak-detection parameters objectively
  instead of by hand-tuning.
when_to_use_negative:
- You already have a manually validated, instrument-specific parameter set — although
  optimisation can still be rerun, the skill targets objective selection rather than
  refining user-specified values
- The samples available for optimisation are the same ones used for classifier training
  — the README explicitly warns to 'choose different samples as we do not want the
  parameters to be optimized using the same data the classifier was trained on'
- The data is not GC-MS raw chromatography-mass spectrometry input (e.g. already-processed
  peak tables or LC-MS-only workflows outside WiPP's scope)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: WiPP
  role: Implements the exhaustive grid search over centWave/matchedFilter parameters,
    applies the SVM classifier and scoring function to each candidate set, and writes
    optimized_parameters.yaml
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which the WiPP preprocessing workflow and its optimisation step
    are built
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:052063c99c7174153787821695d6260bbb89de85057d9e93d15ebb857d38c2be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gcms-peak-detection-parameter-optimization

## Summary

Unsupervised, classifier-driven grid-search optimisation of embedded peak-picking algorithm parameters (centWave, matchedFilter) in WiPP, where every candidate parameter set's peaks are classified by a trained SVM and scored so the best set is selected without manual input.

## When to use

Use when preprocessing GC-MS metabolomics data (mzML/mzData/NetCDF) with WiPP and needing to select centWave or matchedFilter peak-detection parameters objectively instead of by hand-tuning.

## When NOT to use

- You already have a manually validated, instrument-specific parameter set — although optimisation can still be rerun, the skill targets objective selection rather than refining user-specified values
- The samples available for optimisation are the same ones used for classifier training — the README explicitly warns to 'choose different samples as we do not want the parameters to be optimized using the same data the classifier was trained on'
- The data is not GC-MS raw chromatography-mass spectrometry input (e.g. already-processed peak tables or LC-MS-only workflows outside WiPP's scope)

## Inputs

- GC-MS raw sample files (mzML, mzData, or NetCDF) placed in the Optimization_samples directory (copies of samples distinct from Training_samples)
- Project config.yaml containing the candidate peak-detection parameter grid and paths to Training_samples/Optimization_samples
- Algorithm-specific trained SVM peak-quality classifiers (e.g. example_classifier referenced in the classifier block of config.yaml)

## Outputs

- optimized_parameters.yaml containing the optimal centWave and matchedFilter parameter sets (argmax of quality score)
- Scored peak classifications per candidate parameter set, retained for the subsequent full-dataset peak picking step

## How to apply

Place a distinct set of samples (different from the classifier Training_samples) in the Optimization_samples directory, then run the 'pp' stage of run_WiPP.sh, which executes peak detection with centWave and matchedFilter over every combination in the candidate parameter grid defined via config.yaml. Each detected peak is classified by the algorithm-specific trained SVM into one of seven peak-quality classes, and a scoring function rewards peaks in high-quality classes (A–C) while penalising noise and intermediate classes, yielding an aggregate score per parameter set. For each algorithm, the parameter set with the maximum score (argmax of the classifier-driven quality score) is selected and written to optimized_parameters.yaml, with the scored peak classifications retained for the full-dataset peak picking step. The rationale is that optimisation is performed on samples disjoint from classifier training data, preventing overfitting of both the classifier and the detection parameters.

## Related tools

- **WiPP** (Implements the exhaustive grid search over centWave/matchedFilter parameters, applies the SVM classifier and scoring function to each candidate set, and writes optimized_parameters.yaml) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which the WiPP preprocessing workflow and its optimisation step are built)

## Examples

```
cd projects/example_project && ../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The selected parameter set must be the argmax of the classifier-driven quality score across the full candidate grid for each algorithm
- For the chosen parameter set, detected peaks should show the best aggregate class distribution under the A–C-rewarding scoring function (high-quality classes A–C rewarded, noise and intermediate classes penalised)
- Optimisation samples must be disjoint from Training_samples; the pipeline config paths in config.yaml should confirm different files are used
- optimized_parameters.yaml must be produced containing parameter sets for each enabled peak-picking algorithm before the full-dataset peak picking step runs
- Reusing the trained classifier across projects with the same instrument/data acquisition method should reproduce consistent parameter selections

## Limitations

- Optimisation samples must differ from the classifier Training_samples; using the same data biases both classifier and parameter selection
- Accuracy of the optimisation depends on classifier quality — with very few annotated peaks (e.g. the 25-peak example project) the classifier 'is likely not to be accurate'
- WiPP was tested on Ubuntu 16 and CentOS 7.6 only; Ubuntu 18 is not supported due to lacking support of incorporated R packages
- Config must explicitly enable each peak-picking algorithm in config.yaml (the example project runs only one algorithm by default)

## Evidence

- [other] During the optimisation step, the peaks detected under every single candidate parameter set are classified using the algorithm-specific classifier and scored using the scoring function, which enables parameter sets to be evaluated without manual input.: [evidence span withheld — anchor status: not_found]
- [readme] Optimization samples must differ from training samples.: "just make sure to choose different samples as we do not want the parameters to be optimized using the same data the classifier was trained on"
- [readme] Optimisation substep runs within the peak picking stage.: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
- [abstract] Seven-class machine learning quality evaluation underpins the scoring.: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [readme] Classifier can be reused across projects.: "Once trained, the SVM classifiers can be reused on other datasets generated with the same instrument and data acquisition method."
