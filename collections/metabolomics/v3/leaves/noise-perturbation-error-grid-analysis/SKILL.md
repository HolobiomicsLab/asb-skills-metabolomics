---
name: noise-perturbation-error-grid-analysis
description: Decision support for determining at which added-error levels simulated
  single-tracer or multi-tracer isotopologue datasets remain usable for moiety model
  selection, e.g. when extending robustness evaluation beyond sigma in {0.001, 0.01,
  0.1}.
when_to_use_negative:
- The input is experimentally measured (not simulated) isotopologue data where the
  true generating model is unknown, so no selection success rate can be tabulated
- The goal is isotopologue deconvolution alone on a single dataset rather than characterizing
  how added-error magnitude affects model selection
- Data are 13C single-tracer profiles handled by existing MAIMS-style pipelines without
  any model-selection question
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: moiety_modeling
  role: CLI framework used to optimize moiety model parameters, analyze optimizations,
    and rank candidate models by AICc at each sigma level
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Combined simulated annealing and genetic algorithm optimizer used to solve
    the boundary-value inverse fitting problem for each noisy dataset
- name: scipy
  role: Provides additional optimization methods used alongside SAGA-optimize for
    parameter fitting
  repo: https://github.com/scipy/scipy
- name: pickle
  role: Used for saving Python optimization objects in a JSON-serializable form and
    outputting to files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:9a1007201e36b907e1dfe2d2b0735c96b564bfb98d271396dc061b24a6c151d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAGA-optimize
  - moiety_modeling
  - pickle
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# noise-perturbation-error-grid-analysis

## Summary

Systematically perturbs simulated isotopologue timecourses with added normal error across a grid of standard-deviation (sigma) values, then re-runs moiety model optimization and AICc-based selection at each level to quantify how propagated error degrades model-selection usability for single- and multi-tracer MS isotopologue data.

## When to use

Decision support for determining at which added-error levels simulated single-tracer or multi-tracer isotopologue datasets remain usable for moiety model selection, e.g. when extending robustness evaluation beyond sigma in {0.001, 0.01, 0.1}.

## When NOT to use

- The input is experimentally measured (not simulated) isotopologue data where the true generating model is unknown, so no selection success rate can be tabulated
- The goal is isotopologue deconvolution alone on a single dataset rather than characterizing how added-error magnitude affects model selection
- Data are 13C single-tracer profiles handled by existing MAIMS-style pipelines without any model-selection question

## Inputs

- Simulated isotopologue timecourses as dataset JSON files (single-tracer and multi-tracer)
- Candidate moiety model definitions as model JSON files
- Sigma grid of added normal-error standard deviations (dense grid extending beyond {0.001, 0.01, 0.1})

## Outputs

- Saved optimization result files (pickle-serialized Python objects from moiety_modeling)
- Per-condition analysis results from 'analyze optimizations'
- AICc model rankings per sigma condition with generating-model selection outcome
- Aggregated per-sigma selection success rates, e.g. model_selection_success_vs_sigma.csv

## How to apply

Define candidate moiety models and simulate isotopologue timecourses for both single-tracer and multi-tracer cases as model and dataset JSON files. Perturb the simulated isotopologue profiles with added normal error at each standard deviation on a dense sigma grid extending beyond {0.001, 0.01, 0.1}, because thresholding and renormalization in multi-tracer datasets amplify error (sigma = 0.1 drove propagated error to roughly nine times the original signal). For every candidate-model x sigma condition, fit parameters with 'python3 -m moiety_modeling modeling' using --repetition=100, --split, --multiprocess and --energyFunction=logDifference, combining SAGA-optimize (simulated annealing plus genetic algorithm) with scipy optimizers. Save optimization results (via pickle/jsonpickle-serializable Python objects), analyze them with 'python3 -m moiety_modeling analyze optimizations', then rank candidate models per condition with --rankCriteria=AICc and record whether the generating model ranks first. Aggregate per-sigma selection success rates for single- and multi-tracer cases into a table such as model_selection_success_vs_sigma.csv, validating that the noise-free limit matches correct-model selection.

## Related tools

- **moiety_modeling** (CLI framework used to optimize moiety model parameters, analyze optimizations, and rank candidate models by AICc at each sigma level) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Combined simulated annealing and genetic algorithm optimizer used to solve the boundary-value inverse fitting problem for each noisy dataset)
- **scipy** (Provides additional optimization methods used alongside SAGA-optimize for parameter fitting) — https://github.com/scipy/scipy
- **pickle** (Used for saving Python optimization objects in a JSON-serializable form and outputting to files)

## Examples

```
python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- For every sigma in the grid, the AICc-ranked selection outcome for the generating model is tabulated as a success rate
- At the noise-free limit (sigma -> 0), the generating model must rank first, matching correct-model selection
- Per-sigma success rates for single- and multi-tracer cases are aggregated into model_selection_success_vs_sigma.csv
- Each candidate model x sigma condition is fit with --repetition=100 using SAGA-optimize plus scipy optimizers with --energyFunction=logDifference
- Propagated error at high sigma should reflect the paper's expectation (e.g. ~9x the original signal at sigma = 0.1 for multi-tracer data due to thresholding and renormalization)

## Limitations

- Multi-tracer simulated datasets become practically unusable at high added-error levels: with sigma = 0.1 added normal error, propagated error reaches roughly nine times the original signal on average due to thresholding and renormalization
- Data-quality requirements of model selection and validation of multiple isotope tracing model optimization and selection were explicitly left as future work in the original paper
- Selection of incorrect moiety models prevents deconvolution results from being accurately interpreted in terms of relative metabolic flux

## Evidence

- [discussion] Future work will explore data quality requirements of model selection: "Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection."
- [methods] AICc ranking command: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [discussion] Correct model selection requirement: "The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux."
- [readme] Package capabilities: "'moiety_modeling' package provides facilities for moiety model representation, model optimization and model selection."
