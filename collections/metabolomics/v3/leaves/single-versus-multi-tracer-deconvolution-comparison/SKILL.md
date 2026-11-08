---
name: single-versus-multi-tracer-deconvolution-comparison
description: 'Decision support for choosing between single- and multi-tracer SIRM
  isotopologue designs: when you need to know at what added-error standard deviation
  (σ) model selection via AICc ranking breaks down, or must assess data-quality requirements
  before measuring multi-tracer datasets.'
when_to_use_negative:
- Data are real measured isotopologue profiles rather than simulated timecourses —
  this skill evaluates data-quality requirements on simulated data, not deconvolution
  of experimental measurements
- The metabolic system involves only 13C single-tracer data with an already-settled
  model — the comparison of single- vs multi-tracer robustness is not informative
- Flux analysis requires well-curated metabolic networks (e.g., EMU-based 13C MFA)
  — moiety modeling is designed for cases where network models are incomplete or multiple
  deconvolution models are plausible
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: moiety_modeling
  role: Runs model optimization ('modeling'), analysis of optimizations ('analyze
    optimizations'), and AICc-based model ranking ('analyze rank') for each simulated
    σ condition
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Solves the boundary-value inverse problem of parameter fitting via combined
    simulated annealing and genetic algorithm inside each optimization run
- name: scipy
  role: Provides additional optimization methods used alongside SAGA-optimize for
    fitting moiety model parameters
  repo: https://github.com/scipy/scipy
- name: pickle
  role: Used for saving Python optimization objects in a JSON serializable form and
    outputting to files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:924c0f1085f68ea37940a6ef90fa3e3070fbe14a730c6280dac72e83f334932d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - moiety_modeling
  - pickle
  - scipy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-versus-multi-tracer-deconvolution-comparison

## Summary

A simulation-based robustness evaluation that compares how well the moiety_modeling framework selects the correct generating model from single-tracer versus multi-tracer simulated isotopologue timecourses as added-error magnitude (σ) increases, quantifying error propagation through thresholding and renormalization.

## When to use

Decision support for choosing between single- and multi-tracer SIRM isotopologue designs: when you need to know at what added-error standard deviation (σ) model selection via AICc ranking breaks down, or must assess data-quality requirements before measuring multi-tracer datasets.

## When NOT to use

- Data are real measured isotopologue profiles rather than simulated timecourses — this skill evaluates data-quality requirements on simulated data, not deconvolution of experimental measurements
- The metabolic system involves only 13C single-tracer data with an already-settled model — the comparison of single- vs multi-tracer robustness is not informative
- Flux analysis requires well-curated metabolic networks (e.g., EMU-based 13C MFA) — moiety modeling is designed for cases where network models are incomplete or multiple deconvolution models are plausible

## Inputs

- Candidate moiety model definitions as JSON files
- Simulated isotopologue timecourse datasets as JSON files (single-tracer and multi-tracer)
- Sigma grid of added normal error standard deviations (e.g., extending beyond {0.001, 0.01, 0.1})
- Saved optimization result files (pickle/JSON-serialized objects)

## Outputs

- model_selection_success_vs_sigma.csv with per-sigma selection success rates for single- and multi-tracer cases
- AICc rank results indicating whether the generating model ranked first per condition
- Analysis of propagated error magnitudes (e.g., ~9x signal amplification at σ = 0.1 for multi-tracer data)

## How to apply

Define candidate moiety models and simulate isotopologue timecourses as model and dataset JSON files for both single- and multi-tracer conditions. Perturb the simulated isotopologue profiles with added normal error on a dense σ grid extending beyond {0.001, 0.01, 0.1}. For each candidate model × σ condition, run 'python3 -m moiety_modeling modeling' with --repetition=100, --split, --multiprocess and --energyFunction=logDifference, fitting parameters with SAGA-optimize plus scipy optimizers; then analyze optimizations and rank models with 'python3 -m moiety_modeling analyze rank --rankCriteria=AICc'. Record whether the generating model ranks first, and aggregate per-σ selection success rates for single- and multi-tracer cases into model_selection_success_vs_sigma.csv. The rationale is that thresholding and renormalization of isotopologue fractions amplify added error — at σ = 0.1, propagated error reaches roughly nine times the original signal in multi-tracer datasets, making them practically unusable.

## Related tools

- **moiety_modeling** (Runs model optimization ('modeling'), analysis of optimizations ('analyze optimizations'), and AICc-based model ranking ('analyze rank') for each simulated σ condition) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Solves the boundary-value inverse problem of parameter fitting via combined simulated annealing and genetic algorithm inside each optimization run)
- **scipy** (Provides additional optimization methods used alongside SAGA-optimize for fitting moiety model parameters) — https://github.com/scipy/scipy
- **pickle** (Used for saving Python optimization objects in a JSON serializable form and outputting to files)

## Examples

```
python3 -m moiety_modeling modeling --repetition=100 --split --multiprocess --energyFunction=logDifference; python3 -m moiety_modeling analyze optimizations; python3 -m moiety_modeling analyze rank --rankCriteria=AICc
```

## Evaluation signals

- For every σ in the grid, the AICc-ranked selection outcome for the generating model is tabulated as a success rate
- The noise-free limit of the σ grid must match correct-model selection (success rate consistent with noise-free performance)
- Per-condition results reproduce the expected monotonic degradation of multi-tracer usability with increasing σ, with roughly 9x propagated error at σ = 0.1
- Aggregated CSV contains separate success-rate columns/rows for single-tracer and multi-tracer cases across the full σ grid
- Each candidate model × σ condition was run with --repetition=100 and fitted with SAGA-optimize plus scipy optimizers before ranking

## Limitations

- Multi-tracer simulated datasets become practically unusable at high added-error levels: with σ = 0.1 added normal error, propagated error reaches roughly nine times the original signal on average due to thresholding and renormalization
- Conclusions derive from simulated datasets with added normal error; validation of multiple isotope tracing model optimization and selection remained future work in the source article
- Selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux — errors in selection invalidate flux interpretation

## Evidence

- [discussion] Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection.: "Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection."
- [methods] python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [abstract] SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool: "SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool"
- [methods] pickle_ for saving Python objects in a JSON serializable form and outputting to a file.: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [readme] moiety_modeling package provides facilities for moiety model representation, model optimization and model selection.: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
