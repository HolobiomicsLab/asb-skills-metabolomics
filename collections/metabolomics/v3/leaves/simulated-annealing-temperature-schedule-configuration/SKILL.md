---
name: simulated-annealing-temperature-schedule-configuration
description: Use when optimizing boundary-value inverse problems such as moiety model
  parameters, where genetic algorithm operators must be combined with an annealing
  temperature schedule to balance exploration and convergence.
when_to_use_negative:
- The task is standard, well-behaved optimization where the scipy library's additional
  optimization methods suffice rather than a combined annealing-genetic search.
- The input is not a boundary-value inverse problem with defined moiety model parameters,
  e.g., data already deconvoluted or requiring no parameter fitting.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3520
tools:
- name: SAGA-optimize
  role: Python package implementing the combined simulated annealing and genetic algorithm
    used to optimize moiety model parameters
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: moiety_modeling
  role: Invokes the SAGA optimization via the modeling command with logDifference
    energy function over configured repetitions and modes
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Provides additional optimization methods that can complement SAGA optimization
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the optimized moiety state Python objects into a JSON output file
  repo: https://github.com/jsonpickle/jsonpickle
- name: pickle
  role: Used for saving Python objects in a JSON serializable form and outputting
    to a file
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:2635370ec5377ae70845f6d670ae9f5e0da54adb6ac4252b36fbe450d8b91808
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# simulated-annealing-temperature-schedule-configuration

## Summary

Configure the SAGA-optimize combined simulated annealing and genetic algorithm by integrating the annealing temperature schedule into the mutation step, attenuating mutation intensity as temperature drops to optimize moiety model parameters.

## When to use

Use when optimizing boundary-value inverse problems such as moiety model parameters, where genetic algorithm operators must be combined with an annealing temperature schedule to balance exploration and convergence.

## When NOT to use

- The task is standard, well-behaved optimization where the scipy library's additional optimization methods suffice rather than a combined annealing-genetic search.
- The input is not a boundary-value inverse problem with defined moiety model parameters, e.g., data already deconvoluted or requiring no parameter fitting.

## Inputs

- model definition JSON file for the moiety model
- dataset JSON file of MS-measured isotopologue profiles (e.g., 13C isotopologue timecourses of UDP-GlcNAc)
- optimization settings JSON file (population size, crossover rate, mutation rate, temperature schedule, repetitions, energy function)

## Outputs

- optimized moiety state results serialized as a JSON output file (via jsonpickle)
- minimized logDifference objective values across optimization repetitions

## How to apply

Encode the moiety model parameters as Guess/ElementDescription objects representing candidate moiety state values and initialize a Population of candidate solutions with a configurable population size. Implement the SAGA class so that the simulated annealing process is incorporated directly into the mutation step, with configurable crossover rate, mutation rate, and temperature schedule; as the annealing temperature drops, the level of mutation is attenuated, shifting the search from exploration to convergence. Define the energy function (e.g., logDifference for isotopologue profile fitting) and run the optimization via the moiety_modeling modeling command with the model definition JSON, dataset JSON, and optimization settings JSON files, using settings such as 100 repetitions and split/multiprocess modes. Serialize the optimized moiety state results with jsonpickle into a JSON output file for later analysis and ranking.

## Related tools

- **SAGA-optimize** (Python package implementing the combined simulated annealing and genetic algorithm used to optimize moiety model parameters) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **moiety_modeling** (Invokes the SAGA optimization via the modeling command with logDifference energy function over configured repetitions and modes) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Provides additional optimization methods that can complement SAGA optimization) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the optimized moiety state Python objects into a JSON output file) — https://github.com/jsonpickle/jsonpickle
- **pickle** (Used for saving Python objects in a JSON serializable form and outputting to a file)

## Examples

```
python3 -m moiety_modeling modeling --modelFile model.json --dataFile data.json --optFile settings.json --mode multiprocess
```

## Evaluation signals

- The logDifference objective function is minimized over the optimization repetitions (100 repetitions configured in the UDP-GlcNAc run).
- Recovered moiety state values are consistent with the rounded values reported in Table 6 of the paper.
- Mutation levels attenuate monotonically as the annealing temperature drops according to the configured temperature schedule.
- The optimization output JSON can be deserialized back into Python objects via jsonpickle without loss of the moiety state structure.

## Limitations

- SAGA behavior depends on configurable parameters (population size, crossover rate, mutation rate, temperature schedule) that must be set appropriately for the problem.
- Boundary-value inverse problem optimization is stochastic; results may vary across repetitions and require sufficient repetitions to converge.
- Future work on data quality requirements of model selection and validation of multiple isotope tracing model optimization remains outstanding.

## Evidence

- [abstract] SAGA integrates annealing into mutation: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [other] Annealing in mutation step: [evidence span withheld — anchor status: not_found]
- [methods] Pickle-based serialization: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [abstract] Additional scipy optimizers: "Additional optimization methods from the Python scipy library are utilized as well"
