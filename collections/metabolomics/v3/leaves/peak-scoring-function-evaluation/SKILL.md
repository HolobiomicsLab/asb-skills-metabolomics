---
name: peak-scoring-function-evaluation
description: 'Use when optimising GC-MS peak-picking parameters (e.g. centWave or
  matchedFilter) without ground truth: each candidate parameter set''s detected peaks
  can be classified by a trained peak-quality SVM and ranked by an aggregate quality
  score.'
when_to_use_negative:
- The optimisation samples are the same as the Training_samples used to fit the classifier
  — the README warns parameters must not be optimised on the data the classifier was
  trained on
- Ground-truth annotated peaks are available, making a direct supervised comparison
  of parameter sets possible instead of classifier-driven scoring
- The input is a pre-existing peak table rather than raw mzML/mzData/NetCDF chromatograms
  — scoring is applied to peaks freshly detected per candidate parameter set
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: WiPP
  role: 'Implements the grid-search peak-picking parameter optimisation: per-candidate-set
    peak detection, seven-class SVM peak classification, and the A–C-rewarding scoring
    function'
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which the WiPP workflow and its scoring/classification pipeline
    are built
  repo: https://github.com/bihealth/WiPP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:12b5dee48de85671d5f9c14deb79c9c33f9a26ce2d615c3590ab98dd818f8001
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# peak-scoring-function-evaluation

## Summary

A scoring function that aggregates classifier-assigned peak-quality classes (A–C rewarded, noise and intermediate classes penalised) into a per-parameter-set score, enabling unsupervised, manual-input-free evaluation of peak-detection parameter sets in WiPP's grid search.

## When to use

Use when optimising GC-MS peak-picking parameters (e.g. centWave or matchedFilter) without ground truth: each candidate parameter set's detected peaks can be classified by a trained peak-quality SVM and ranked by an aggregate quality score.

## When NOT to use

- The optimisation samples are the same as the Training_samples used to fit the classifier — the README warns parameters must not be optimised on the data the classifier was trained on
- Ground-truth annotated peaks are available, making a direct supervised comparison of parameter sets possible instead of classifier-driven scoring
- The input is a pre-existing peak table rather than raw mzML/mzData/NetCDF chromatograms — scoring is applied to peaks freshly detected per candidate parameter set

## Inputs

- GC-MS raw files in mzML, mzData, or NetCDF format placed in the Optimization_samples directory (different samples from those used for classifier training)
- Project config.yaml with paths to Optimization_samples and peak-picking algorithm settings
- Trained algorithm-specific SVM peak-quality classifier (seven-class)

## Outputs

- Per-parameter-set aggregate quality scores across the exhaustive grid search
- optimized_parameters.yaml containing the optimal centWave and matchedFilter parameter sets
- Retained scored peak classifications for the subsequent full-dataset peak picking step

## How to apply

Run peak detection on a dedicated Optimization_samples subset (distinct from classifier Training_samples) for every combination in the candidate parameter grid, using both centWave and matchedFilter. Classify every detected peak with the algorithm-specific trained SVM, which assigns one of seven peak-quality classes. Apply the scoring function that rewards peaks in high-quality classes A–C and penalises noise and intermediate classes, aggregating a single score per parameter set. Select, per algorithm, the parameter set with the maximum score (argmax over the exhaustive grid) and write the optimal centWave and matchedFilter parameters to optimized_parameters.yaml for the subsequent full-dataset peak picking. The rationale is that class distributions of peaks are a proxy for peak-picking quality, so the score replaces manual parameter tuning.

## Related tools

- **WiPP** (Implements the grid-search peak-picking parameter optimisation: per-candidate-set peak detection, seven-class SVM peak classification, and the A–C-rewarding scoring function) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which the WiPP workflow and its scoring/classification pipeline are built) — https://github.com/bihealth/WiPP

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The selected parameter set must be the argmax of the classifier-driven quality score over the exhaustive candidate grid
- The winning parameter set's detected peaks must show the best aggregate class distribution under the A–C-rewarding scoring function
- optimized_parameters.yaml must contain one optimal parameter set per enabled algorithm (centWave and matchedFilter)
- Optimisation must use samples disjoint from the classifier's Training_samples; overlap invalidates the score

## Limitations

- Scoring quality depends on the trained SVM classifier; with few annotated peaks (e.g. only 25 per algorithm in the example project) the classifier is likely inaccurate
- The classifier is instrument/protocol specific — a trained classifier can be reused only on datasets from the same instrument and acquisition method
- The exhaustive grid search over parameter combinations is computationally expensive, since peaks are detected and classified for every candidate set

## Evidence

- [full_text] During the optimisation step, the peaks detected under every single candidate parameter set are classified using the algorithm-specific classifier and scored using the scoring function, which enables parameter sets to be evaluated without manual input.: [evidence span withheld — anchor status: not_found]
- [readme] The Optimization_samples directory is similar to the Training_samples directory, just make sure to choose different samples as we do not want the parameters to be optimized using the same data the classifier was trained on.: "just make sure to choose different samples as we do not want the parameters to be optimized using the same data the classifier was trained on"
- [abstract] WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes.: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [readme] Many sequential substeps are in fact happening during this final step of the pipeline such as classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset.: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset"
- [readme] Peak annotation usually takes several hours (1200 peaks per algorithm by default), but you only need to do that once per instrument/protocol.: "Once trained, the SVM classifiers can be reused on other datasets generated with the same instrument and data acquisition method."
