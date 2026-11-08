---
name: isotopologue-deconvolution-model-selection
description: Use when MS-measured isotopologue timecourses (single or multiple isotope
  tracers, e.g. 13C UDP-GlcNAc SIRM data) must be deconvoluted into moiety abundances
  and the -fitting deconvolution model must be chosen from several plausible models.
when_to_use_negative:
- Input data are raw metabolic network models requiring full 13C MFA — the EMU-style
  flux analysis framework, not moiety modeling, is the appropriate tool when curated
  networks are available
- Only 13C single-tracer data without any need for model selection — the earlier MAIMS
  software handles that simpler case
- The input is already deconvoluted moiety abundance data rather than measured isotopologue
  profiles
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: moiety_modeling
  role: Core package for moiety model representation, optimization, analysis, ranking
    and plotting
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Combined simulated annealing and genetic algorithm optimizer for the boundary-value
    inverse problem
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Supplementary optimization methods applied after SAGA
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes optimized moiety state objects to JSON output files
  repo: https://github.com/jsonpickle/jsonpickle
- name: matplotlib
  role: Visualization of optimized moiety and isotopologue curves
  repo: https://github.com/matplotlib/matplotlib
- name: docopt
  role: Command-line interface parsing for the moiety_modeling commands
  repo: https://github.com/docopt/docopt
- name: GAIMS
  role: Prototype Perl isotopologue deconvolution program used as a comparison benchmark
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:6ba2cfcc7c421b7b3a2b7c80bcdcd3b39a6912ee02f6a022a9737222710a28b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - GAIMS
  - SAGA-optimize
  - docopt
  - jsonpickle
  - matplotlib
  - moiety_modeling
  - numpy
  - pickle
  - scipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# isotopologue-deconvolution-model-selection

## Summary

A workflow for deconvolving mass spectrometry isotopologue profiles into moiety abundances using the moiety_modeling Python package, and for selecting among plausible moiety models via AICc ranking after SAGA (simulated annealing + genetic algorithm) parameter optimization.

## When to use

Use when MS-measured isotopologue timecourses (single or multiple isotope tracers, e.g. 13C UDP-GlcNAc SIRM data) must be deconvoluted into moiety abundances and the best-fitting deconvolution model must be chosen from several plausible models.

## When NOT to use

- Input data are raw metabolic network models requiring full 13C MFA — the EMU-style flux analysis framework, not moiety modeling, is the appropriate tool when curated networks are available
- Only 13C single-tracer data without any need for model selection — the earlier MAIMS software handles that simpler case
- The input is already deconvoluted moiety abundance data rather than measured isotopologue profiles

## Inputs

- Moiety model definition JSON file (one per candidate model)
- Isotopologue dataset JSON file (e.g. 13C UDP-GlcNAc timecourses from LnCaP-LN3 cells)
- Optimization settings JSON file (energy function, repetition count, split/multiprocess options)

## Outputs

- Optimized moiety state parameter values (JSON, serialized via jsonpickle/pickle)
- Analysis results JSON with fitting statistics per model
- AICc-based model rank order across candidate models
- Plots of fitted isotopologue fraction / moiety curves (matplotlib)

## How to apply

Encode candidate moiety models as JSON model definitions alongside dataset JSON and optimization-settings JSON files, then run moiety_modeling modeling with SAGA-optimize (combined simulated annealing and genetic algorithm, annealing integrated into the mutation step) supplemented by scipy optimizers, e.g. with --repetition=100 --split --multiprocess. Analyze the optimized results with 'analyze optimizations', then rank candidate models with 'analyze rank --rankCriteria=AICc' to select the model describing relative isotope fluxes. Visualize fitted moiety/isotopologue curves with the 'plot moiety' command backed by numpy and matplotlib, and serialize optimization objects with pickle/jsonpickle. The default logDifference energy function is appropriate for standard fitting; an absolute-difference objective can be substituted to reproduce GAIMS-comparable behavior, though that objective tends to overfit and may prevent AICc from selecting the expert-derived model.

## Related tools

- **moiety_modeling** (Core package for moiety model representation, optimization, analysis, ranking and plotting) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Combined simulated annealing and genetic algorithm optimizer for the boundary-value inverse problem) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Supplementary optimization methods applied after SAGA) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes optimized moiety state objects to JSON output files) — https://github.com/jsonpickle/jsonpickle
- **matplotlib** (Visualization of optimized moiety and isotopologue curves) — https://github.com/matplotlib/matplotlib
- **docopt** (Command-line interface parsing for the moiety_modeling commands) — https://github.com/docopt/docopt
- **GAIMS** (Prototype Perl isotopologue deconvolution program used as a comparison benchmark)

## Examples

```
python3 -m moiety_modeling modeling --models=<model_jsonfile> --datasets=<dataset_jsonfile> --optimizations=<optimizationSetting_json> --repetition=100 --split --multiprocess; python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- Reproduced AICc rank order of UDP-GlcNAc models must match the rank comparison reported in Additional file 9
- Optimized moiety state parameter values must be consistent with the rounded values reported in Table 6 and Additional file 11
- Fitted isotopologue fraction curves must visually match the parameter graphs in Additional file 10
- logDifference objective must be minimized across the 100 optimization repetitions; multiple repetitions should converge to comparable energies

## Limitations

- The AICc model selection fails to select the expert-derived model when using the absolute-difference objective because of overfitting with that objective function
- Results are not identical between moiety_modeling and GAIMS even with the same objective function, due to implementation and stochastic differences
- MAIMS-style predecessor tools handle only 13C single-isotope tracer data and lack model selection; moiety_modeling was developed to fill this gap
- Future work is still needed on data quality requirements of model selection and validation of multiple-isotope model optimization

## Evidence

- [methods] moiety_modeling command usage: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [abstract] abstract tooling: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [readme] readme scope: "'moiety_modeling' package provides facilities for moiety model representation, model optimization and model selection."
- [results] reproduction of GAIMS comparison: [evidence span withheld — anchor status: not_found]
- [discussion] discussion caveat: "Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection."
