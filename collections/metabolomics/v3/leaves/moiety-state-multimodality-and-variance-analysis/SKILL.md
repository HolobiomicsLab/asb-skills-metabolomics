---
name: moiety-state-multimodality-and-variance-analysis
description: Use when repeated optimizations of simulated single- vs multi-tracer
  SIRM isotopologue datasets exist at equivalent added normal error and you must quantify
  how propagated error alters moiety state value distributions.
when_to_use_negative:
- Data are raw mass spectrometry isotopologue profiles that have not yet been deconvolved
  or fitted by moiety model optimization
- Goal is model selection/ranking (use AICc rank criteria) rather than characterizing
  the spread or modality of optimized parameter values
- Only a single optimization run per dataset exists, so no across-repetition variance
  or multimodality can be assessed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: moiety_modeling
  role: loads datasets/models, runs repeated SAGA optimization, and aggregates optimized
    moiety state values via 'analyze optimizations'
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: combined simulated annealing and genetic algorithm solver used for 5000-step
    repeated optimization of moiety model parameters
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: jsonpickle
  role: serializes each repetition's optimized moiety state parameters to JSON files
  repo: https://github.com/jsonpickle/jsonpickle
- name: numpy
  role: computes distribution statistics (variance, skewness) and histogram binning
    of aggregated moiety state values
  repo: https://github.com/numpy/numpy
- name: matplotlib
  role: renders histograms contrasting single-tracer versus multi-tracer moiety state
    value distributions
  repo: https://github.com/matplotlib/matplotlib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:8952044b0869ca702855cb3f1b2a68b273d2eaf9a95e20c09a2eee4081bb3c0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - jsonpickle
  - matplotlib
  - moiety_modeling
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# moiety-state-multimodality-and-variance-analysis

## Summary

Aggregates optimized moiety state parameters across repeated SAGA optimization runs to compare propagated-error effects—variance, skewness, and multimodality—on acetyl and uracil moiety state value distributions between simulated multi-tracer and single-tracer isotopologue datasets.

## When to use

Use when repeated optimizations of simulated single- vs multi-tracer SIRM isotopologue datasets exist at equivalent added normal error and you must quantify how propagated error alters moiety state value distributions.

## When NOT to use

- Data are raw mass spectrometry isotopologue profiles that have not yet been deconvolved or fitted by moiety model optimization
- Goal is model selection/ranking (use AICc rank criteria) rather than characterizing the spread or modality of optimized parameter values
- Only a single optimization run per dataset exists, so no across-repetition variance or multimodality can be assessed

## Inputs

- simulated single-tracer isotopologue dataset JSON files
- simulated multi-tracer isotopologue dataset JSON files
- moiety model JSON definitions (e.g. acetyl and uracil moiety models)
- optimization settings JSON
- per-repetition optimized moiety state parameter JSON files (jsonpickle-serialized)

## Outputs

- aggregated moiety state value distributions across 100 SAGA repetitions per dataset
- numpy/matplotlib histograms contrasting single-tracer vs multi-tracer moiety state value distributions at equivalent noise sigma

## How to apply

Load the simulated single- and multi-tracer isotopologue dataset JSON files with their moiety model JSON definitions and optimization settings via moiety_modeling. Run repeated SAGA-optimize (combined simulated annealing and genetic algorithm) at 5000 steps per repetition, e.g. 100 repetitions with --repetition=100 --split --multiprocess and --energyFunction=logDifference. Persist each repetition's optimized moiety state parameters to JSON files using jsonpickle, then aggregate the acetyl and uracil moiety state values per dataset with 'python3 -m moiety_modeling analyze optimizations'. Finally, build histograms with numpy and matplotlib overlaying single-tracer versus multi-tracer distributions at matched noise sigma to reveal variance/skewness inflation in multi-tracer runs and bimodality in single-tracer runs.

## Related tools

- **moiety_modeling** (loads datasets/models, runs repeated SAGA optimization, and aggregates optimized moiety state values via 'analyze optimizations') — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (combined simulated annealing and genetic algorithm solver used for 5000-step repeated optimization of moiety model parameters) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **jsonpickle** (serializes each repetition's optimized moiety state parameters to JSON files) — https://github.com/jsonpickle/jsonpickle
- **numpy** (computes distribution statistics (variance, skewness) and histogram binning of aggregated moiety state values) — https://github.com/numpy/numpy
- **matplotlib** (renders histograms contrasting single-tracer versus multi-tracer moiety state value distributions) — https://github.com/matplotlib/matplotlib

## Examples

```
python3 -m moiety_modeling modeling --repetition=100 --split --multiprocess --energyFunction=logDifference; python3 -m moiety_modeling analyze optimizations --a optimizationPaths.txt
```

## Evaluation signals

- Multi-tracer moiety state value distributions show wider variance than single-tracer distributions at equivalent added normal error sigma
- Multi-tracer distributions exhibit some additional skewness relative to single-tracer distributions at comparable propagated error
- Single-tracer moiety state value distributions show pronounced multimodality (e.g. bimodality) at comparable propagated error
- 100 optimization repetitions per dataset completed, each persisted as a jsonpickle JSON file of optimized moiety state parameters
- Qualitative pattern matches Fig 7 of the paper (wider variance/skewness in multi-tracer; bimodality in single-tracer)

## Limitations

- Propagated error in multi-tracer analyses widens variance and introduces skewness, so optimized moiety state values carry more uncertainty than single-tracer estimates
- Single-tracer distributions can be bimodal, meaning single representative optimized values may be misleading
- Future work is required to establish the data quality requirements of model selection and to validate multiple-isotope tracing model optimization

## Evidence

- [results] propagated error leads to wider variances and some additional skewness in multi-tracer moiety state values: [evidence span withheld — anchor status: not_found]
- [abstract] SAGA-optimize boundary-value inverse problem: [evidence span withheld — anchor status: not_found]
- [methods] analyze optimizations workflow: "python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>"
- [methods] jsonpickle for saving objects: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [discussion] future work on data quality requirements: "Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection."
