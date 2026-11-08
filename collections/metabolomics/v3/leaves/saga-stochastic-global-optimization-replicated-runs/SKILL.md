---
name: saga-stochastic-global-optimization-replicated-runs
description: Use when fitting a stochastic global optimizer (SAGA) to isotopologue/moiety
  model parameters and you need parameter uncertainty, i.e. distributions over repeated
  runs rather than one -fit value.
when_to_use_negative:
- The data are raw MS isotopologue profiles not yet represented as a moiety model
  — model definition must precede SAGA optimization.
- The goal is model selection/ranking across candidate moiety models — use the AICc
  ranking workflow ('analyze rank --rankCriteria=AICc') instead of replicated single-model
  optimization.
- The input is a complete metabolic network for EMU-based 13C MFA — this moiety modeling
  approach was developed because such curated-network methods are incomplete.
tools:
- name: SAGA-optimize
  role: Combined simulated annealing + genetic algorithm optimizer used to solve the
    boundary-value inverse problem; runs 5000 steps per repetition
- name: moiety_modeling
  role: Package providing moiety model representation and the CLI ('modeling', 'analyze
    optimizations') that orchestrates replicated SAGA runs and aggregation
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: jsonpickle
  role: Serializes each repetition's optimized moiety state parameter objects to JSON
    files for later aggregation
  repo: https://github.com/jsonpickle/jsonpickle
- name: scipy
  role: Provides additional optimization methods available alongside SAGA within the
    framework
  repo: https://github.com/scipy/scipy
- name: matplotlib
  role: Renders histograms contrasting single-tracer vs multi-tracer optimized moiety
    state distributions
  repo: https://github.com/matplotlib/matplotlib
- name: numpy
  role: Computes variance/skewness summaries and histogram binning of aggregated moiety
    state values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:fc0803d7881749bd6fe5ba41c7ebe5072fd656c690025f5ef0800a8309983ea7
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# saga-stochastic-global-optimization-replicated-runs

## Summary

Run replicated SAGA-optimize (combined simulated annealing + genetic algorithm) fits of a moiety model to MS isotopologue data, persisting every repetition so that optimized parameter distributions reveal uncertainty and multimodality rather than a single point estimate.

## When to use

Use when fitting a stochastic global optimizer (SAGA) to isotopologue/moiety model parameters and you need parameter uncertainty, i.e. distributions over repeated runs rather than one best-fit value.

## When NOT to use

- The data are raw MS isotopologue profiles not yet represented as a moiety model — model definition must precede SAGA optimization.
- The goal is model selection/ranking across candidate moiety models — use the AICc ranking workflow ('analyze rank --rankCriteria=AICc') instead of replicated single-model optimization.
- The input is a complete metabolic network for EMU-based 13C MFA — this moiety modeling approach was developed because such curated-network methods are incomplete.

## Inputs

- Simulated or measured single-/multi-tracer isotopologue dataset JSON files
- Moiety model definition JSON files
- Optimization settings JSON (including --repetition, --split, --multiprocess, --energyFunction parameters)

## Outputs

- Per-repetition optimized moiety state parameter files serialized with jsonpickle
- Aggregated optimization results from 'moiety_modeling analyze optimizations'
- Histograms of optimized moiety state value distributions (numpy/matplotlib)

## How to apply

Load the isotopologue dataset JSON files together with the moiety model JSON definitions and optimization settings via the moiety_modeling package, then execute optimization with 'python3 -m moiety_modeling modeling' using --repetition=100 --split --multiprocess and --energyFunction=logDifference, letting SAGA-optimize (simulated annealing followed by genetic algorithm) run 5000 steps per repetition. The rationale is that SAGA solves the non-linear deconvolution as a boundary-value inverse problem, but each stochastic run converges to a slightly different optimum, so repeated runs are needed to sample the parameter distribution. Persist each repetition's optimized moiety state parameters to JSON with jsonpickle, then aggregate across the 100 repetitions per dataset with the moiety_modeling analyze-optimizations command ('python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>'). Finally summarize the optimized moiety state values (e.g. acetyl and uracil moiety state distributions) with numpy/matplotlib histograms and compare variances, skewness, and modality across conditions at equivalent added normal error.

## Related tools

- **SAGA-optimize** (Combined simulated annealing + genetic algorithm optimizer used to solve the boundary-value inverse problem; runs 5000 steps per repetition)
- **moiety_modeling** (Package providing moiety model representation and the CLI ('modeling', 'analyze optimizations') that orchestrates replicated SAGA runs and aggregation) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **jsonpickle** (Serializes each repetition's optimized moiety state parameter objects to JSON files for later aggregation) — https://github.com/jsonpickle/jsonpickle
- **scipy** (Provides additional optimization methods available alongside SAGA within the framework) — https://github.com/scipy/scipy
- **matplotlib** (Renders histograms contrasting single-tracer vs multi-tracer optimized moiety state distributions) — https://github.com/matplotlib/matplotlib
- **numpy** (Computes variance/skewness summaries and histogram binning of aggregated moiety state values)

## Examples

```
python3 -m moiety_modeling modeling --repetition=100 --split --multiprocess --energyFunction=logDifference
```

## Evaluation signals

- Each repetition completes the full SAGA schedule (5000 optimization steps) and produces a serialized parameter file; with --repetition=100 there must be 100 output files per dataset.
- Aggregated multi-tracer moiety state value distributions show wider variance and additional skewness than single-tracer distributions at equivalent added normal error sigma.
- Single-tracer optimized moiety state distributions at comparable propagated error show pronounced multimodality/bimodality, matching the qualitative pattern of Fig 7 of the paper.
- The energy function used is logDifference across all repetitions, so aggregated distributions are comparable across datasets.
- Optimized moiety state values stay within their model-defined bounds (SAGA handles boundary-value constraints) with no out-of-range parameters in the serialized files.

## Limitations

- Replicated stochastic optimization is computationally expensive; multiprocessing and splitting are used to make 100 repetitions feasible.
- Propagated error in multi-tracer datasets widens and skews parameter distributions, so point-estimate comparisons at equivalent noise are qualitative, not exact.
- The authors note that future work must explore data quality requirements of model selection and validate multiple-isotope tracing model optimization and selection.

## Evidence

- [abstract] SAGA solves boundary-value inverse problem with simulated annealing and genetic algorithm: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [methods] moiety_modeling optimizes moiety model parameters: "Using moiety_modeling to optimize parameters of moiety model."
- [methods] analyze optimizations CLI aggregates results: "python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>"
- [readme] moiety_modeling provides model optimization facilities: "'moiety_modeling' package provides facilities for moiety model representation, model optimization and model selection."
