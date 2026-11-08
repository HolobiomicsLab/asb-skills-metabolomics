---
name: saga-hybrid-optimization-implementation
description: Use when optimizing parameters of moiety models against mass spectrometry
  isotopologue timecourses (e.g., 13C UDP-GlcNAc data) where the nonlinear deconvolution
  problem requires a hybrid GA/annealing search rather than a single optimizer.
when_to_use_negative:
- The isotopologue profile is already deconvoluted or the analysis goal is only model
  ranking/plotting rather than parameter optimization
- A smooth, unconstrained objective where simpler scipy optimizers suffice — the article
  notes scipy optimization methods are utilized as well for such cases
- Data are 13C single-tracer metabolites already handled by MAIMS-style tools without
  a model-selection requirement
edam_operation: http://edamontology.org/operation_2425
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: moiety_modeling
  role: Front-end that loads model/dataset/settings JSON, invokes SAGA optimization
    with the logDifference energy function, and serializes results
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Implements the combined simulated annealing and genetic algorithm with annealing
    embedded in the mutation step
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Additional optimization methods used alongside SAGA for model optimization
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes optimized moiety state results (Python objects) into JSON output
    files
  repo: https://github.com/jsonpickle/jsonpickle
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:ec5f1ee41254c7b2f5662cf3350516b6966ec5782e266ac5f9de846db9e15c8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - jsonpickle
  - moiety_modeling
  - pickle
  - scipy
  edam_operation: http://edamontology.org/operation_2425
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# saga-hybrid-optimization-implementation

## Summary

Reproduce the SAGA-optimize combined simulated annealing and genetic algorithm, which integrates the annealing schedule into the mutation operator to solve the boundary-value inverse problem of moiety model parameter optimization for MS isotopologue deconvolution.

## When to use

Use when optimizing parameters of moiety models against mass spectrometry isotopologue timecourses (e.g., 13C UDP-GlcNAc data) where the nonlinear deconvolution problem requires a hybrid GA/annealing search rather than a single optimizer.

## When NOT to use

- The isotopologue profile is already deconvoluted or the analysis goal is only model ranking/plotting rather than parameter optimization
- A smooth, unconstrained objective where simpler scipy optimizers suffice — the article notes scipy optimization methods are utilized as well for such cases
- Data are 13C single-tracer metabolites already handled by MAIMS-style tools without a model-selection requirement

## Inputs

- Moiety model definition JSON file
- MS isotopologue dataset JSON file (e.g., 13C UDP-GlcNAc timecourse from LnCaP-LN3 cells)
- Optimization settings JSON file (population size, crossover rate, mutation rate, temperature schedule, repetitions)
- Guess/ElementDescription objects encoding candidate moiety state values

## Outputs

- JSON output file of optimized moiety state values (jsonpickle-serialized)
- Optimization results files consumed by moiety_modeling analyze/rank commands
- logDifference energy values per optimization repetition

## How to apply

Encode moiety model parameters as Guess/ElementDescription objects holding candidate moiety state values and initialize a Population of candidate solutions with a configurable population size. Implement the SAGA class so that simulated annealing is folded directly into the mutation step: the mutation magnitude attenuates as the annealing temperature drops, while crossover and mutation operators run at configurable crossover and mutation rates under a temperature schedule. Drive the search with the logDifference energy function, run 100 repetitions in split and multiprocess modes via the moiety_modeling modeling command, and load the model definition JSON, dataset JSON, and optimization settings JSON for the UDP-GlcNAc expert model. Serialize optimized moiety state results into a JSON output file with jsonpickle for downstream analyze and rank steps.

## Related tools

- **moiety_modeling** (Front-end that loads model/dataset/settings JSON, invokes SAGA optimization with the logDifference energy function, and serializes results) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Implements the combined simulated annealing and genetic algorithm with annealing embedded in the mutation step) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Additional optimization methods used alongside SAGA for model optimization) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes optimized moiety state results (Python objects) into JSON output files) — https://github.com/jsonpickle/jsonpickle

## Examples

```
python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>
```

## Evaluation signals

- The logDifference objective is minimized across optimization repetitions
- Recovered moiety state values are consistent with the rounded values reported in Table 6 of the paper for the UDP-GlcNAc expert model
- Output JSON files are valid jsonpickle-serialized structures loadable by moiety_modeling analyze
- The annealing temperature schedule monotonically attenuates mutation magnitude over generations
- Downstream ranking with --rankCriteria=AICc selects the expected optimal moiety model

## Limitations

- SAGA-optimize targets boundary-value inverse problems; its hybrid GA/annealing is tuned to nonlinear moiety deconvolution rather than general optimization
- Future work must validate multiple isotope tracing model optimization and selection, and data quality requirements of model selection remain unexplored
- Correct moiety model selection is prerequisite: deconvolution results are only accurately interpretable in terms of relative metabolic flux when the correct model is selected

## Evidence

- [abstract] SAGA combined GA/annealing purpose: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [full_text] Annealing integrated into mutation: [evidence span withheld — anchor status: not_found]
- [methods] Optimization workflow entry point: "Using moiety_modeling to optimize parameters of moiety model."
- [methods] JSON-serializable result output: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [readme] Package capabilities: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
