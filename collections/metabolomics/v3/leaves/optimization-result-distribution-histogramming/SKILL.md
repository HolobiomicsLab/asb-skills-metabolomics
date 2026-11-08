---
name: optimization-result-distribution-histogramming
description: Use when many optimization repetitions of the same moiety model have
  been persisted (e.g. 100 SAGA repetitions via jsonpickle) and you need to compare
  the spread, skewness, or multimodality of optimized moiety state values across datasets
  at equivalent added error.
when_to_use_negative:
- When a single optimization run per dataset suffices (e.g. only ranking models by
  AICc) — no distribution over repetitions exists to histogram.
- When the goal is visualizing fitted isotopologue profiles rather than parameter
  uncertainty — use 'python3 -m moiety_modeling plot moiety <analysisResults_file>'
  instead.
- When input data are experimental isotopologue timecourses without repeated optimization
  ensembles; comparing propagated error requires simulated datasets at controlled
  noise levels.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0153
tools:
- name: moiety_modeling
  role: Executes the repeated SAGA optimization and aggregates optimized moiety state
    parameters across repetitions via the analyze-optimizations command
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Provides the combined simulated annealing and genetic algorithm used for each
    of the 5000-step optimization repetitions whose results are histogrammed
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: jsonpickle
  role: Serializes each repetition's optimized moiety state parameters to JSON files
    for later aggregation
  repo: https://github.com/jsonpickle/jsonpickle
- name: numpy
  role: Computes variance/skewness summaries and binning for moiety state value histograms
  repo: https://github.com/numpy/numpy
- name: matplotlib
  role: Renders histograms overlaying single-tracer versus multi-tracer moiety state
    distributions at equivalent noise sigma
  repo: https://github.com/matplotlib/matplotlib
- name: scipy
  role: Supports statistics on the aggregated optimization result distributions
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:3d6794a30924bd31041f120d03637aa7b42cd7ad29bfb898a4a1896a939a43e7
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# optimization-result-distribution-histogramming

## Summary

Aggregates repeated SAGA optimization runs of moiety models and visualizes the distributions of optimized moiety state values (e.g. acetyl and uracil moiety abundances) as histograms, to assess how propagated measurement error reshapes parameter estimates in multi- versus single-tracer isotopologue datasets.

## When to use

Use when many optimization repetitions of the same moiety model have been persisted (e.g. 100 SAGA repetitions via jsonpickle) and you need to compare the spread, skewness, or multimodality of optimized moiety state values across datasets at equivalent added error.

## When NOT to use

- When a single optimization run per dataset suffices (e.g. only ranking models by AICc) — no distribution over repetitions exists to histogram.
- When the goal is visualizing fitted isotopologue profiles rather than parameter uncertainty — use 'python3 -m moiety_modeling plot moiety <analysisResults_file>' instead.
- When input data are experimental isotopologue timecourses without repeated optimization ensembles; comparing propagated error requires simulated datasets at controlled noise levels.

## Inputs

- JSON files of optimized moiety state parameters, one per repetition (saved with jsonpickle)
- Text file listing optimization result paths (--a <optimizationPaths_txtfile>)
- Simulated single-tracer and multi-tracer isotopologue dataset JSON files with equivalent added normal error
- Moiety model JSON definitions and optimization settings JSON

## Outputs

- Aggregated moiety state value distributions across 100 repetitions per dataset
- Histograms (numpy/matplotlib) of acetyl and uracil moiety state values contrasting single- vs multi-tracer results
- Per-moiety statistics: variance, skewness, and multimodality indicators (e.g. bimodality of single-tracer distributions)

## How to apply

Run repeated parameter optimization (e.g. 100 repetitions of SAGA-optimize, 5000 steps each, with --split --multiprocess and --energyFunction=logDifference) so each dataset yields an ensemble of optimized moiety state parameters, saved per repetition as JSON via jsonpickle. Aggregate these optimization results across repetitions with 'python3 -m moiety_modeling analyze-optimizations --a <optimizationPaths_txtfile>' to collect the optimized acetyl/uracil moiety state values per dataset. Then build histograms of the state-value distributions with numpy and matplotlib, overlaying single-tracer versus multi-tracer results at the same noise sigma and comparable propagated error. Decision point: interpret wider variance and additional skewness in multi-tracer distributions as the signature of error propagation through the extra tracer dimension, and pronounced bimodality in single-tracer distributions as evidence of multiple near-optimal solutions.

## Related tools

- **moiety_modeling** (Executes the repeated SAGA optimization and aggregates optimized moiety state parameters across repetitions via the analyze-optimizations command) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Provides the combined simulated annealing and genetic algorithm used for each of the 5000-step optimization repetitions whose results are histogrammed) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **jsonpickle** (Serializes each repetition's optimized moiety state parameters to JSON files for later aggregation) — https://github.com/jsonpickle/jsonpickle
- **numpy** (Computes variance/skewness summaries and binning for moiety state value histograms) — https://github.com/numpy/numpy
- **matplotlib** (Renders histograms overlaying single-tracer versus multi-tracer moiety state distributions at equivalent noise sigma) — https://github.com/matplotlib/matplotlib
- **scipy** (Supports statistics on the aggregated optimization result distributions) — https://github.com/scipy/scipy

## Examples

```
python3 -m moiety_modeling analyze-optimizations --a optimizationPaths.txt
```

## Evaluation signals

- Multi-tracer moiety state value distributions show wider variance and some additional skewness than single-tracer distributions at equivalent added normal error.
- Single-tracer moiety state value distributions show pronounced bimodality (multimodality) at comparable propagated error.
- Each dataset was optimized with exactly 100 independent SAGA repetitions (5000 steps each) before aggregation, so histograms are built from ≥100 samples per moiety state.
- Histogram patterns are consistent with the qualitative comparison reported in Fig 7 of the paper.
- All repetition outputs load correctly from their jsonpickle JSON files before aggregation (no missing/corrupt repetition files).

## Limitations

- Conclusions about propagated error depend on simulated datasets with controlled added normal error; experimental data quality requirements for model selection remain unvalidated.
- Bimodal single-tracer distributions indicate multiple near-optimal solutions, so a single best-fit moiety state value may be misleading without the repetition ensemble.
- The comparison requires equivalent added error and comparable propagated error between the multi- and single-tracer datasets; mismatched noise levels confound the variance/skewness interpretation.

## Evidence

- [results] Comparing multi-tracer and single-tracer experiments with equivalent added normal error: [evidence span withheld — anchor status: not_found]
- [results] single-tracer bimodality: [evidence span withheld — anchor status: not_found]
- [methods] optimization invocation: [evidence span withheld — anchor status: not_found]
- [methods] aggregation command: "python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>"
- [abstract] SAGA purpose: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
