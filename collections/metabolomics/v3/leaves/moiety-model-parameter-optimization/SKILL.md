---
name: moiety-model-parameter-optimization
description: Use when fitting parameters of a moiety model to MS-measured isotopologue
  timecourse data (e.g. 13C UDP-GlcNAc data) to solve the boundary-value inverse problem
  before model ranking with AICc.
when_to_use_negative:
- Input is raw MS spectra rather than pre-computed isotopologue fraction timecourses
  usable as dataset JSON files
- The metabolite is a simple single-moiety case better handled by MAIMS-style single-tracer
  analysis without model optimization
- A well-curated metabolic network exists and full metabolic flux analysis via the
  EMU framework is more appropriate
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: moiety_modeling
  role: Package providing moiety model representation, model optimization and model
    selection; the 'modeling' command drives SAGA optimization of moiety model parameters
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Combined simulated annealing and genetic algorithm optimizer solving the boundary-value
    inverse problem; annealing is integrated into the mutation step
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Supplementary optimization methods used to refine SAGA-optimized model parameters
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes optimized moiety state results (Python objects) to JSON output
    files
  repo: https://github.com/jsonpickle/jsonpickle
- name: docopt
  role: Command-line interface parsing for the moiety_modeling modeling/analyze/plot
    subcommands
  repo: https://github.com/docopt/docopt
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:0245de8a10aa5baa7a5639584d95dff99f4bed8d2f38e32114be45a1f92a6be7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - docopt
  - jsonpickle
  - matplotlib
  - moiety_modeling
  - numpy
  - pickle
  - scipy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# moiety-model-parameter-optimization

## Summary

Optimizes parameters of moiety models deconvoluting MS isotopologue profiles using the SAGA-optimize combined simulated annealing and genetic algorithm, supplemented by scipy optimizers, within the moiety_modeling package.

## When to use

Use when fitting parameters of a moiety model to MS-measured isotopologue timecourse data (e.g. 13C UDP-GlcNAc data) to solve the boundary-value inverse problem before model ranking with AICc.

## When NOT to use

- Input is raw MS spectra rather than pre-computed isotopologue fraction timecourses usable as dataset JSON files
- The metabolite is a simple single-moiety case better handled by MAIMS-style single-tracer analysis without model optimization
- A well-curated metabolic network exists and full metabolic flux analysis via the EMU framework is more appropriate

## Inputs

- moiety model JSON file (model definitions, e.g. UDP-GlcNAc acetyl/uracil moiety models)
- isotopologue dataset JSON file (MS-measured isotopologue timecourses)
- optimization settings JSON file (SAGA temperature schedule, crossover/mutation rates, 5000 steps, 100 repetitions)

## Outputs

- optimization result files with optimized moiety state parameters serialized via jsonpickle/pickle to JSON
- aggregated analysis results JSON from 'analyze optimizations' (fit quality, moiety state distributions)
- optimized parameter values comparable to rounded values in Table 6 of the paper

## How to apply

Encode the moiety model parameters as Guess/ElementDescription objects (candidate moiety state values) and initialize a Population of candidate solutions with a configurable population size. Run SAGA-optimize, which integrates simulated annealing directly into the mutation step of a genetic algorithm, attenuating mutation levels as the annealing temperature drops, together with additional scipy optimization methods. Execute 'python3 -m moiety_modeling modeling --models=<model_jsonfile> --datasets=<dataset_jsonfile> --optimizations=<optimizationSetting_json> --repetition=100 --split --multiprocess' with the desired energy function (default logDifference; an absolute-difference objective can be substituted to reproduce GAIMS behavior). Run 5000 SAGA steps per repetition and persist optimized moiety state values via jsonpickle/pickle to JSON output files. Aggregate and analyze repetitions with 'python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>'.

## Related tools

- **moiety_modeling** (Package providing moiety model representation, model optimization and model selection; the 'modeling' command drives SAGA optimization of moiety model parameters) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Combined simulated annealing and genetic algorithm optimizer solving the boundary-value inverse problem; annealing is integrated into the mutation step) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Supplementary optimization methods used to refine SAGA-optimized model parameters) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes optimized moiety state results (Python objects) to JSON output files) — https://github.com/jsonpickle/jsonpickle
- **docopt** (Command-line interface parsing for the moiety_modeling modeling/analyze/plot subcommands) — https://github.com/docopt/docopt

## Examples

```
python3 -m moiety_modeling modeling --models=<model_jsonfile> --datasets=<dataset_jsonfile> --optimizations=<optimizationSetting_json> --repetition=100 --split --multiprocess
```

## Evaluation signals

- The logDifference energy function is minimized across repetitions, and recovered moiety state values match the rounded values reported in Table 6 of the paper
- Optimized parameter values and AICc rank order are reasonably comparable to GAIMS results in Additional files 9-11 when an absolute-difference objective is used (results need not be identical due to implementation and stochastic differences)
- Across 100 repetitions, optimized moiety state distributions are tight (single-tracer) with expected wider variance/skewness for multi-tracer datasets at equivalent added error
- With clean (noise-free) simulated data, repeated optimizations support selection of the generating model by AICc
- Every repetition's optimized parameters serialize to valid JSON via jsonpickle without loss of the Guess/ElementDescription objects

## Limitations

- Optimization is stochastic: reproducing GAIMS with an absolute-difference objective gives results that are not identical due to implementation and stochastic differences, and neither method can select the expert-derived model by AICc under that objective because of overfitting
- Propagated error degrades optimization quality: with sigma = 0.1 added normal error on multi-tracer datasets, propagated error reaches roughly nine times the original signal due to thresholding and renormalization
- Single-tracer moiety state value distributions can be bimodal across repetitions at comparable propagated error, complicating interpretation of point estimates

## Evidence

- [supplementary] SAGA annealing-mutation integration: [evidence span withheld — anchor status: not_found]
- [methods] modeling command: "python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>"
- [abstract] SAGA purpose: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [methods] pickle serialization: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [readme] package scope: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
