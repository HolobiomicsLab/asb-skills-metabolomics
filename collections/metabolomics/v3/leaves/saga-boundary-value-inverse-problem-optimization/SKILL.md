---
name: saga-boundary-value-inverse-problem-optimization
description: Use when fitting a nonlinear moiety/deconvolution model to mass spectrometry
  isotopologue profiles (e.g. UDP-GlcNAc 13C timecourses) requires solving a boundary-value
  inverse problem where local optimizers are unreliable and many perturbation models
  must be compared.
when_to_use_negative:
- Isotopologue data come from a single well-characterized metabolite and only point
  estimates of moiety abundances are needed without model selection among competing
  models
- The analysis requires a fully curated metabolic network for metabolic flux analysis
  (e.g. EMU framework use cases) rather than deconvolution of isotopologue profiles
  into moiety abundances
- Input is 13C single-tracer data already handled by MAIMS and no model comparison
  is required
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SAGA-optimize
  role: Solves the boundary-value inverse problem via a combined simulated annealing
    and genetic algorithm; the optimizer invoked by moiety_modeling during parameter
    fitting
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: moiety_modeling
  role: Command-line driver that sets up models/datasets/settings JSON, runs SAGA
    optimization with --energyFunction=logDifference over --repetition=100, and consolidates
    results
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Provides additional optimization methods utilized alongside SAGA for model
    parameter optimization
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the ranked analysis results (Python objects) to JSON files after
    optimization and ranking
  repo: https://github.com/jsonpickle/jsonpickle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:3d951f2eeaca2c6843946a9283456e6acd550587b69fcea3974acdf40669a2d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAGA-optimize
  - jsonpickle
  - moiety_modeling
  - scipy
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# saga-boundary-value-inverse-problem-optimization

## Summary

Optimization of moiety model parameters for isotopologue deconvolution by delegating the boundary-value inverse problem to SAGA-optimize, a Python package combining simulated annealing and genetic algorithm, as implemented in the moiety_modeling workflow for 13C (and multi-tracer) MS timecourses.

## When to use

Use when fitting a nonlinear moiety/deconvolution model to mass spectrometry isotopologue profiles (e.g. UDP-GlcNAc 13C timecourses) requires solving a boundary-value inverse problem where local optimizers are unreliable and many perturbation models must be compared.

## When NOT to use

- Isotopologue data come from a single well-characterized metabolite and only point estimates of moiety abundances are needed without model selection among competing models
- The analysis requires a fully curated metabolic network for metabolic flux analysis (e.g. EMU framework use cases) rather than deconvolution of isotopologue profiles into moiety abundances
- Input is 13C single-tracer data already handled by MAIMS and no model comparison is required

## Inputs

- models JSON file containing expert-derived and perturbation moiety models (e.g. 6_G1R1A1U3 and 40 perturbations)
- datasets JSON file with MS-measured isotopologue timecourse measurements (e.g. UDP-GlcNAc 13C isotopologues at 34 h, 48 h, 72 h)
- optimization settings JSON file (repetitions, energy function, optimizer options)

## Outputs

- Optimization result files with fitted moiety model parameters and objective scores per repetition
- An analysis JSON file (saved via jsonpickle) consolidating fitted results and AICc rankings across models

## How to apply

Assemble a models JSON file (e.g. the expert-derived moiety model 6_G1R1A1U3 plus perturbation models), a datasets JSON file with isotopologue measurements at each timepoint (34 h, 48 h, 72 h), and an optimization settings JSON file. Run parameter optimization with 'python3 -m moiety_modeling modeling' setting --repetition=100 and --energyFunction=logDifference, with --split and --multiprocess for scaling, which delegates the boundary-value inverse problem to the SAGA (simulated annealing plus genetic algorithm) optimizer from SAGA-optimize; additional optimization methods from the Python scipy library can also be utilized. Repeat optimization across all candidate models so fitted results can be consolidated via 'analyze optimizations' and compared. The rationale is that SAGA's combined stochastic search avoids local minima in the nonlinear deconvolution landscape, and AICc ranking over the repeated fits selects the model best supported by the timecourse data.

## Related tools

- **SAGA-optimize** (Solves the boundary-value inverse problem via a combined simulated annealing and genetic algorithm; the optimizer invoked by moiety_modeling during parameter fitting) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **moiety_modeling** (Command-line driver that sets up models/datasets/settings JSON, runs SAGA optimization with --energyFunction=logDifference over --repetition=100, and consolidates results) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Provides additional optimization methods utilized alongside SAGA for model parameter optimization) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the ranked analysis results (Python objects) to JSON files after optimization and ranking) — https://github.com/jsonpickle/jsonpickle

## Examples

```
python3 -m moiety_modeling modeling --repetition=100 --split --multiprocess --energyFunction=logDifference
```

## Evaluation signals

- With 100 repetitions per model and logDifference energy function, the expert-derived model 6_G1R1A1U3 should rank first by AICc at approximately -229.2918
- Known competitor values should be reproduced, e.g. 6_G1R1A1U3_r4 at AICc ≈ -227.5208 and 7_G2R1A1U3_g5 at AICc ≈ -208.1346
- All 41 models (expert-derived plus 40 perturbations) should appear in the ranked analysis JSON output written via jsonpickle
- Fitted isotopologue profiles should visually match the measured 13C UDP-GlcNAc timecourse at 34 h, 48 h, and 72 h when plotted with 'analyze/plot' outputs

## Limitations

- Future work must explore the data quality requirements of model selection and validate multiple isotope tracing model optimization and selection
- Selection of correct moiety models is required for deconvolution results to be accurately interpreted in terms of relative metabolic flux; a wrongly specified model set invalidates interpretation
- Optimization is stochastic (simulated annealing plus genetic algorithm), so multiple repetitions (e.g. --repetition=100) are needed; single runs may land in local minima

## Evidence

- [abstract] The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [results] Using SAGA optimization with a log difference objective function (100 repetitions per model), the expert-derived moiety model was successfully selected among all 40 moiety models by AICc: [evidence span withheld — anchor status: not_found]
- [methods] python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>: "python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>"
- [abstract] Additional optimization methods from the Python scipy library are utilized as well: "Additional optimization methods from the Python scipy library are utilized as well"
