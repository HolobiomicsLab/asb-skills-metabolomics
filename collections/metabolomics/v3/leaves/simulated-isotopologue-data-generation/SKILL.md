---
name: simulated-isotopologue-data-generation
description: Use when you need to assess how noise magnitude affects moiety model
  selection — e.g., tabulating whether the generating model ranks first by AICc as
  added-error sigma varies, or defining data-quality requirements for simulated 13C
  SIRM isotopologue datasets.
when_to_use_negative:
- Input is real experimental MS isotopologue data to be deconvoluted into moiety abundances,
  rather than simulated data for robustness/model-selection testing.
- The task uses MAIMS-style 13C single-tracer data only and does not involve model
  selection among plausible moiety models.
- A single noise-free fit suffices — no sigma sweep or data-quality requirement analysis
  is needed.
tools:
- name: moiety_modeling
  role: Generates/represents moiety models and simulated isotopologue data (model
    and dataset JSON files); runs parameter optimization ('modeling') and result analysis/ranking
    ('analyze optimizations', 'analyze rank --rankCriteria=AICc').
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Performs parameter optimization of moiety models on simulated data via combined
    simulated annealing and genetic algorithm for the boundary-value inverse problem.
- name: scipy
  role: Supplies additional optimization methods used alongside SAGA-optimize to fit
    moiety model parameters at each sigma condition.
- name: pickle
  role: Saves Python objects (model definitions, optimization results) in JSON serializable
    form for later analysis steps.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:ae8dfc37b1599fd95a1d788a96d4a420b14bf22a2abab63d963a27c31b4e4fe8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAGA-optimize
  - moiety_modeling
  - pickle
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# simulated-isotopologue-data-generation

## Summary

Generates synthetic single- and multi-tracer isotopologue timecourses from candidate moiety models (as model and dataset JSON files) and perturbs them with added normal error across a dense sigma grid, enabling quantitative assessment of model-selection robustness in the moiety modeling framework.

## When to use

Use when you need to assess how noise magnitude affects moiety model selection — e.g., tabulating whether the generating model ranks first by AICc as added-error sigma varies, or defining data-quality requirements for simulated 13C SIRM isotopologue datasets.

## When NOT to use

- Input is real experimental MS isotopologue data to be deconvoluted into moiety abundances, rather than simulated data for robustness/model-selection testing.
- The task uses MAIMS-style 13C single-tracer data only and does not involve model selection among plausible moiety models.
- A single noise-free fit suffices — no sigma sweep or data-quality requirement analysis is needed.

## Inputs

- candidate moiety model definitions (model JSON files)
- simulated isotopologue timecourse profiles for single- and multi-tracer datasets (dataset JSON files)
- grid of added-error standard deviations (sigma), denser than {0.001, 0.01, 0.1}

## Outputs

- perturbed simulated isotopologue profiles per sigma condition
- saved optimization results (pickle/JSON-serializable Python objects)
- per-condition AICc model rankings with success/failure of generating-model selection
- aggregated model_selection_success_vs_sigma.csv (per-sigma selection success rates for single- and multi-tracer cases)

## How to apply

Define candidate moiety models and simulate isotopologue timecourses for single- and multi-tracer datasets, saved as model and dataset JSON files. Perturb the simulated profiles with added normal error at each standard deviation on a denser sigma grid extending beyond the published {0.001, 0.01, 0.1} levels. For each candidate model and sigma condition, run 'python3 -m moiety_modeling modeling' with --repetition=100, --split, --multiprocess and --energyFunction=logDifference, fitting parameters with SAGA-optimize plus scipy optimizers. Analyze saved optimization results, then rank candidate models per condition with --rankCriteria=AICc and record whether the generating model ranks first. Aggregate per-sigma selection success rates for single- vs multi-tracer cases into model_selection_success_vs_sigma.csv; the noise-free limit must recover correct-model selection.

## Related tools

- **moiety_modeling** (Generates/represents moiety models and simulated isotopologue data (model and dataset JSON files); runs parameter optimization ('modeling') and result analysis/ranking ('analyze optimizations', 'analyze rank --rankCriteria=AICc').) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Performs parameter optimization of moiety models on simulated data via combined simulated annealing and genetic algorithm for the boundary-value inverse problem.)
- **scipy** (Supplies additional optimization methods used alongside SAGA-optimize to fit moiety model parameters at each sigma condition.)
- **pickle** (Saves Python objects (model definitions, optimization results) in JSON serializable form for later analysis steps.)

## Examples

```
python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- Noise-free limit validation: at sigma approaching zero, the AICc-ranked selection outcome must recover correct-model selection.
- For every sigma in the grid, the AICc-ranked outcome for the generating model is tabulated as an explicit success rate.
- With --repetition=100 and --split, repeated optimization runs per condition should converge consistently before ranking.
- At sigma = 0.1 with multi-tracer data, expect propagated error to reach roughly nine times the original signal on average (thresholding and renormalization), marking datasets as practically unusable.
- Aggregated CSV (model_selection_success_vs_sigma.csv) must contain per-sigma success rates separated for single- and multi-tracer cases.

## Limitations

- Multi-tracer simulated datasets become practically unusable at high added-error levels: with sigma = 0.1, propagated error reaches roughly nine times the original signal on average due to thresholding and renormalization.
- Data-quality requirements of model selection and validation of multiple isotope tracing model optimization/selection were explicitly left as future work, so sigma-grid conclusions are simulation-based rather than experimentally validated.
- Correct moiety model selection is a prerequisite for deconvolution results interpretable as relative metabolic flux; wrong-model selection at high sigma invalidates downstream flux interpretation.

## Evidence

- [discussion] future data-quality requirements: "Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection."
- [discussion] correct model selection for flux interpretation: "The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux."
- [methods] AICc ranking CLI: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [abstract] package roles: "SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool"
