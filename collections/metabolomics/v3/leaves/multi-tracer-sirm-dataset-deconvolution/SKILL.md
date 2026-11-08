---
name: multi-tracer-sirm-dataset-deconvolution
description: Use when MS isotopologue profiles from single- or multi-isotope SIRM
  experiments must be decomposed into moiety abundances (e.
when_to_use_negative:
- Performing full metabolic flux analysis on a well-curated metabolic network — the
  EMU framework is the established approach there, whereas moiety modeling targets
  deconvolution without requiring complete network models
- Data are already resolved moiety abundances or isotopologue fractions — no deconvolution
  is needed
- The metabolite cannot be decomposed into well-defined moiety units with isotope
  incorporation patterns
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: moiety_modeling
  role: Command-line Python package providing moiety model representation, optimization,
    model selection, and result plotting; invoked as 'python3 -m moiety_modeling'
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Combined simulated annealing and genetic algorithm used for model optimization
    of the boundary-value inverse problem, 5000 steps per repetition
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Additional optimization methods from the Python scipy library are utilized
    as well
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Saving optimized Python objects in a JSON serializable form and outputting
    to a file
  repo: https://github.com/jsonpickle/jsonpickle
- name: matplotlib
  role: Generating histograms/plots of optimized moiety distributions for comparing
    single- vs multi-tracer results
  repo: https://github.com/matplotlib/matplotlib
- name: docopt
  role: Command-line interface parsing for the moiety_modeling package
  repo: https://github.com/docopt/docopt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:5389da9ab9b1b9254a324546cda41c5f5551f16b5e970888b060cee15ef28362
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
  - scipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-tracer-sirm-dataset-deconvolution

## Summary

Deconvolute mass spectrometry measured isotopologue profiles from stable isotope resolved metabolomics (SIRM) datasets using moiety models, supporting both single- and multiple-isotope tracer data via repeated SAGA-optimize parameter estimation. This addresses the gap left by tools like MAIMS, which handle only 13C single-tracer data and lack model selection.

## When to use

Use when MS isotopologue profiles from single- or multi-isotope SIRM experiments must be decomposed into moiety abundances (e.g., acetyl and uracil moieties of a metabolite), when the optimal deconvolution model must be selected from plausible moiety models, or when propagated-error effects on optimized moiety state distributions must be characterized.

## When NOT to use

- Performing full metabolic flux analysis on a well-curated metabolic network — the EMU framework is the established approach there, whereas moiety modeling targets deconvolution without requiring complete network models
- Data are already resolved moiety abundances or isotopologue fractions — no deconvolution is needed
- The metabolite cannot be decomposed into well-defined moiety units with isotope incorporation patterns

## Inputs

- MS-measured isotopologue profile data (single- or multi-tracer SIRM datasets, e.g., 13C/15N timecourses)
- Moiety model JSON definitions describing metabolite decomposition into moieties
- Optimization settings JSON (SAGA-optimize parameters, energy function)
- Simulated isotopologue datasets with added normal error for benchmarking

## Outputs

- Per-repetition optimized moiety state parameters stored as JSON via jsonpickle
- Aggregated analysis results from 'moiety_modeling analyze optimizations'
- AICc-ranked model selection results
- Moiety abundance / isotopologue deconvolution fits and plotted moiety distributions (matplotlib)

## How to apply

Represent the metabolite as a moiety model (e.g., acetyl and uracil moieties of UDP-GlcNAc) in a JSON model definition, then fit it to isotopologue timecourse data by running the moiety_modeling command-line pipeline with the combined simulated annealing + genetic algorithm (SAGA-optimize) for 5000 steps per repetition, using --repetition=100 --split --multiprocess and --energyFunction=logDifference. Persist each repetition's optimized moiety state parameters with jsonpickle so distributions can be aggregated. Aggregate and rank analyses with 'moiety_modeling analyze optimizations' and 'analyze rank' using AICc criteria to select the correct moiety model among plausible ones, since correct model selection is required for interpreting results as relative metabolic flux. For simulated datasets, run the same repeated-optimization protocol at equivalent added normal error on both single- and multi-tracer versions to assess propagated error: multi-tracer fits show wider variance and skewness in moiety state values, while single-tracer distributions can be bimodal.

## Related tools

- **moiety_modeling** (Command-line Python package providing moiety model representation, optimization, model selection, and result plotting; invoked as 'python3 -m moiety_modeling') — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Combined simulated annealing and genetic algorithm used for model optimization of the boundary-value inverse problem, 5000 steps per repetition) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Additional optimization methods from the Python scipy library are utilized as well) — https://github.com/scipy/scipy
- **jsonpickle** (Saving optimized Python objects in a JSON serializable form and outputting to a file) — https://github.com/jsonpickle/jsonpickle
- **matplotlib** (Generating histograms/plots of optimized moiety distributions for comparing single- vs multi-tracer results) — https://github.com/matplotlib/matplotlib
- **docopt** (Command-line interface parsing for the moiety_modeling package) — https://github.com/docopt/docopt

## Examples

```
python3 -m moiety_modeling modeling <model.json> --repetition=100 --split --multiprocess --energyFunction=logDifference
```

## Evaluation signals

- Robust performance confirmed by reproducing isotopologue deconvolution and moiety model selection on 13C isotopologue timecourses of UDP-GlcNAc in human prostate cancer LnCaP-LN3 cells
- Optimized model ranks best under AICc ranking criteria ('analyze rank --rankCriteria=AICc')
- With 100 repetitions, multi-tracer optimized moiety state value distributions show wider variance and skewness relative to single-tracer at equivalent added normal error (expected pattern, cf. Fig 7)
- Single-tracer replicated fits show pronounced bimodality/multimodality at comparable propagated error
- All repetition results persist as loadable JSON objects and complete without failures under --split --multiprocess

## Limitations

- MAIMS, the prior tool in this lineage, handles only 13C single isotope tracer data and does not address model selection; multi-tracer support is the specific advance of moiety_modeling
- Selection of incorrect moiety models yields deconvolution results that cannot be accurately interpreted in terms of relative metabolic flux
- Data quality requirements of model selection and validation of multiple isotope tracing model optimization remain future work
- The framework depends on stochastic SAGA optimization, so repeated runs (e.g., 100 repetitions) are needed to characterize propagated error and multimodality

## Evidence

- [abstract] SAGA-optimize is useful for solving boundary-value inverse problems: "SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool"
- [abstract] robust performance confirmation: "we were able to confirm its robust performance in isotopologue deconvolution and moiety model selection"
- [intro] MAIMS single-tracer limitation: "the MAIMS software handles only 13C single isotope tracer data and does not address model selection"
- [methods] pickle-based persistence of results: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [methods] AICc ranking command: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [discussion] correct model selection requirement: "The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux."
- [readme] installation via pip: "'moiety_modeling' runs under Python 3.6+ and is available through python3-pip."
