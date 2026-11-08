---
name: evolutionary-operator-and-population-design
description: Use when optimizing continuous model parameters (e.g. moiety state values)
  against a non-linear objective like logDifference, where naive scipy optimizers
  are insufficient and a combined simulated annealing / genetic algorithm with tunable
  population and operators is needed.
when_to_use_negative:
- Data are a single-isotope 13C tracer dataset that the existing MAIMS software already
  handles without model selection
- The model is embedded in a well-curated metabolic network amenable to standard 13C
  flux analysis frameworks (EMU) rather than moiety deconvolution
- The optimization target is a simple convex/linear fit already handled by additional
  optimization methods from the Python scipy library
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: moiety_modeling
  role: 'Driver package: moiety model representation, model optimization and model
    selection; invokes SAGA via the modeling command'
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Implements the combined simulated annealing and genetic algorithm (SAGA class,
    Population, Guess/ElementDescription) used for model optimization
- name: scipy
  role: Provides additional optimization methods used alongside SAGA
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Two-way JSON serialization of Python objects (optimized moiety states) to
    output files
  repo: https://github.com/jsonpickle/jsonpickle
- name: pickle
  role: Saving Python objects in a JSON serializable form and outputting to a file
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:5178ef0107f24c1cd716599060b85d20a79e69b741ed89f410ce41a5586efb38
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# evolutionary-operator-and-population-design

## Summary

Design of a hybrid SAGA optimizer that folds simulated annealing into the mutation operator of a genetic algorithm (configurable crossover rate, mutation rate, and temperature schedule) to solve the boundary-value inverse problem of moiety model parameter optimization for MS isotopologue deconvolution.

## When to use

Use when optimizing continuous model parameters (e.g. moiety state values) against a non-linear objective like logDifference, where naive scipy optimizers are insufficient and a combined simulated annealing / genetic algorithm with tunable population and operators is needed.

## When NOT to use

- Data are a single-isotope 13C tracer dataset that the existing MAIMS software already handles without model selection
- The model is embedded in a well-curated metabolic network amenable to standard 13C flux analysis frameworks (EMU) rather than moiety deconvolution
- The optimization target is a simple convex/linear fit already handled by additional optimization methods from the Python scipy library

## Inputs

- Moiety model definition JSON file
- Dataset JSON file (MS-measured isotopologue timecourses, e.g. 13C UDP-GlcNAc in LnCaP-LN3 cells)
- Optimization settings JSON file (population size, crossover rate, mutation rate, temperature schedule, repetitions)
- Energy/objective function (logDifference)

## Outputs

- Optimized moiety state values serialized with jsonpickle into a JSON output file
- Optimization results for downstream analysis, ranking (AICc), and plotting via moiety_modeling

## How to apply

Encode the moiety model parameters as Guess/ElementDescription objects (candidate moiety state values) and initialize a Population of candidate solutions with a configurable population size. Implement the SAGA class so that the annealing process is integrated directly into the mutation step, attenuating the level of mutation as the annealing temperature drops, alongside crossover and mutation operators with configurable crossover rate, mutation rate, and temperature schedule. Run the optimization with the logDifference energy function (e.g. 100 repetitions) in split and multiprocess modes via the moiety_modeling modeling command. Serialize the optimized moiety state results with jsonpickle into a JSON output file. The rationale is that temperature-driven mutation attenuation lets the search start broad and anneal toward precise parameter values while the genetic operators preserve population diversity.

## Related tools

- **moiety_modeling** (Driver package: moiety model representation, model optimization and model selection; invokes SAGA via the modeling command) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Implements the combined simulated annealing and genetic algorithm (SAGA class, Population, Guess/ElementDescription) used for model optimization)
- **scipy** (Provides additional optimization methods used alongside SAGA) — https://github.com/scipy/scipy
- **jsonpickle** (Two-way JSON serialization of Python objects (optimized moiety states) to output files) — https://github.com/jsonpickle/jsonpickle
- **pickle** (Saving Python objects in a JSON serializable form and outputting to a file)

## Examples

```
python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- The logDifference objective is minimized across 100 repetitions of the SAGA run
- Recovered moiety state values match the rounded values reported in Table 6 of the paper for the UDP-GlcNAc expert model
- Optimization logs show mutation levels attenuating as the annealing temperature drops along the schedule
- Output JSON files can be loaded back via jsonpickle and consumed by moiety_modeling analyze/rank/plot commands (e.g. AICc ranking succeeds)

## Limitations

- MAIMS-style prior software handles only 13C single isotope tracer data, motivating the combined-annealing/genetic design for multi-isotope cases
- Simultaneous use of multiple stable isotopes complicates isotopologue profile analysis, increasing the difficulty of operator and schedule tuning
- Future work must explore data quality requirements of model selection and validation of multiple isotope tracing model optimization

## Evidence

- [methods] SAGA-optimize incorporates the annealing process directly into the mutation step, attenuating the level of mutation as the annealing temperature drops: [evidence span withheld — anchor status: not_found]
- [abstract] SAGA-optimize is useful for solving boundary-value inverse problems, and moiety_modeling is an easy-to-use tool for mass spectrometry isotopologue profile deconvolution: [evidence span withheld — anchor status: not_found]
- [methods] pickle_ for saving Python objects in a JSON serializable form and outputting to a file.: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [methods] python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [readme] The moiety_modeling package provides facilities for moiety model representation, model optimization and model selection.: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
