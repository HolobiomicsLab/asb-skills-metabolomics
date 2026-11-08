---
name: exhaustive-grid-search-optimization
description: Use when tuning peak detection parameters for GC-MS metabolomics data
  (mzML/mzData/NetCDF) and no ground-truth peak list exists, so parameter sets must
  be ranked by machine-learning peak-quality scores instead of manual curation.
when_to_use_negative:
- Optimisation samples are the same as the classifier training samples — the README
  explicitly says to choose different samples so parameters are not optimised on classifier
  training data
- Input is already a processed peak table or feature matrix — this skill optimises
  raw chromatogram peak detection, not downstream statistics
- A trusted reference peak list or manual annotation exists that could serve as a
  direct objective function instead of classifier-based scoring
edam_operation: http://edamontology.org/operation_2425
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: WiPP
  role: 'Workflow implementing the exhaustive grid search: runs peak detection per
    candidate parameter set, applies the SVM classifier and quality scoring function,
    and writes optimized_parameters.yaml'
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language in which the WiPP preprocessing workflow, classifier scoring, and
    parameter optimisation are built
  repo: https://github.com/bihealth/WiPP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:80fcbe98cced522bcb048a61ff9a9446258c042c2910f8750b06ef7ae97e5059
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_2425
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# exhaustive-grid-search-optimization

## Summary

Unsupervised, classifier-driven exhaustive grid search over peak-detection algorithm parameters (centWave, matchedFilter) in WiPP, scoring every candidate parameter set by SVM peak-quality classes to select optimal GC-MS peak picking parameters without manual input.

## When to use

Use when tuning peak detection parameters for GC-MS metabolomics data (mzML/mzData/NetCDF) and no ground-truth peak list exists, so parameter sets must be ranked by machine-learning peak-quality scores instead of manual curation.

## When NOT to use

- Optimisation samples are the same as the classifier training samples — the README explicitly says to choose different samples so parameters are not optimised on classifier training data
- Input is already a processed peak table or feature matrix — this skill optimises raw chromatogram peak detection, not downstream statistics
- A trusted reference peak list or manual annotation exists that could serve as a direct objective function instead of classifier-based scoring

## Inputs

- config.yaml project configuration file
- GC-MS raw files (mzML, mzData, or NetCDF) in the Optimization_samples directory
- Trained algorithm-specific SVM peak-quality classifiers (seven peak classes)

## Outputs

- optimized_parameters.yaml with the argmax-scoring centWave and matchedFilter parameter sets
- Scored peak classifications per candidate parameter set, retained for full-dataset peak picking

## How to apply

Load the project config.yaml and the GC-MS files placed in the Optimization_samples directory, ensuring these are different samples from those used for classifier training. Run peak detection with centWave and with matchedFilter over every combination in the candidate parameter grid on the optimisation samples. Classify every detected peak with the algorithm-specific trained SVM classifier, which assigns one of seven peak-quality classes, then apply the scoring function that rewards peaks in high-quality classes (A–C) and penalises noise and intermediate classes to aggregate a score per parameter set. Select, per algorithm, the parameter set with the maximum score across the exhaustive grid search, and write the optimal centWave and matchedFilter parameters to optimized_parameters.yaml for the subsequent full-dataset peak picking step. The rationale is that peak quality — not a reference peak list — becomes the objective function, giving an impartial comparison across embedded peak picking algorithms.

## Related tools

- **WiPP** (Workflow implementing the exhaustive grid search: runs peak detection per candidate parameter set, applies the SVM classifier and quality scoring function, and writes optimized_parameters.yaml) — https://github.com/bihealth/WiPP
- **Python 3** (Language in which the WiPP preprocessing workflow, classifier scoring, and parameter optimisation are built) — https://github.com/bihealth/WiPP

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The selected parameters are the argmax of the classifier-driven quality score across the full candidate grid — verify no other parameter set in the grid has a higher aggregate score
- Per parameter set, the aggregate class distribution should show peaks concentrated in high-quality classes (A–C) for the chosen parameters, with noise/intermediate classes penalised
- Optimization_samples must differ from Training_samples; check the config paths point to disjoint sample copies
- optimized_parameters.yaml exists and contains optimal parameter sets for both centWave and matchedFilter before the full-dataset peak picking step runs

## Limitations

- Parameter quality depends on classifier accuracy; with very few annotated peaks (e.g. the 25-peak example project) the classifier is likely inaccurate, and WiPP provides a pre-trained classifier per instrument/protocol as a workaround
- Only optimises the embedded peak picking algorithms (centWave, matchedFilter); algorithms must be explicitly enabled in config.yaml (the example project enables only one to speed up testing)
- Exhaustive grid search over all combinations is computationally heavy; core count must be tuned with the inline -n <CORES> parameter

## Evidence

- [results] During the optimisation step, the peaks detected under every single candidate parameter set are classified using the algorithm-specific classifier and scored using the scoring function, which enables parameter sets to be evaluated without manual input.: [evidence span withheld — anchor status: not_found]
- [abstract] WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes.: "WiPP evaluates the quality of detected peaks using a machine learning-based classification scheme based on seven peak classes."
- [readme] The Optimization_samples directory is similar to the Training_samples directory, just make sure to choose different samples as we do not want the parameters to be optimized using the same data the classifier was trained on.: "make sure to choose different samples as we do not want the parameters to be optimized using the same data the classifier was trained on"
- [methods] peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration: "peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
