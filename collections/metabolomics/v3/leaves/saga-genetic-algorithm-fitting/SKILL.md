---
name: saga-genetic-algorithm-fitting
description: Use when deconvoluting MS-measured isotopologue profiles (single or multiple
  isotope tracers, e.g. 13C UDP-GlcNAc timecourses) into moiety abundances, or when
  fitting parameters of a moiety model that requires stochastic global optimization.
when_to_use_negative:
- The data require network-based metabolic flux analysis using well-curated metabolic
  networks (e.g. the EMU framework) rather than isotopologue deconvolution of a specific
  metabolite.
- Input is 13C-only single-isotope tracer data and no model selection is needed —
  the MAIMS software already handles that simpler case.
- Exact deterministic reproduction of another tool's output is required (e.g. bit-identical
  GAIMS results) — stochastic SA/GA fitting is not appropriate for that goal.
tools:
- name: SAGA-optimize
  role: Core optimizer solving the boundary-value inverse problem through combined
    simulated annealing and genetic algorithm
- name: moiety_modeling
  role: Package providing moiety model representation, optimization entry points,
    analysis, ranking, and plotting commands
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Supplementary optimization methods applied after SAGA to refine fitted parameters
  repo: https://github.com/scipy/scipy
- name: pickle
  role: Saving Python optimization objects in a JSON serializable form and outputting
    to a file
- name: matplotlib
  role: Visualization of fitted moiety results via 'plot moiety'
  repo: https://github.com/matplotlib/matplotlib
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:ee6a006df7ea914033998665ca00dcb2b8111719f982c0ff4e599bfc9121cc11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - matplotlib
  - moiety_modeling
  - numpy
  - pickle
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# saga-genetic-algorithm-fitting

## Summary

Fit moiety-model parameters to mass spectrometry isotopologue timecourses using SAGA-optimize, a combined simulated annealing and genetic algorithm for boundary-value inverse problems, optionally supplemented by scipy optimizers, then analyze and rank fitted models by AICc.

## When to use

Use when deconvoluting MS-measured isotopologue profiles (single or multiple isotope tracers, e.g. 13C UDP-GlcNAc timecourses) into moiety abundances, or when fitting parameters of a moiety model that requires stochastic global optimization.

## When NOT to use

- The data require network-based metabolic flux analysis using well-curated metabolic networks (e.g. the EMU framework) rather than isotopologue deconvolution of a specific metabolite.
- Input is 13C-only single-isotope tracer data and no model selection is needed — the MAIMS software already handles that simpler case.
- Exact deterministic reproduction of another tool's output is required (e.g. bit-identical GAIMS results) — stochastic SA/GA fitting is not appropriate for that goal.

## Inputs

- moiety model definition JSON file
- dataset JSON file (isotopologue timecourse data, e.g. 13C UDP-GlcNAc from LnCaP-LN3 cells)
- optimization settings JSON file (energy function choice, optimizer configuration)
- text file listing paths to optimization results (for analysis)

## Outputs

- optimized parameter values per moiety model (saved as-serializable Python objects via pickle/jsonpickle)
- analysis results JSON file of fitted isotopologue fraction curves
- AICc-based model ranking across candidate moiety models
- visual plots of fitted moiety curves (matplotlib figures)

## How to apply

Load the moiety model JSON, dataset JSON, and optimization-settings JSON into the moiety_modeling package, choosing the energy function deliberately: the default logDifference for standard fitting, or an absolute-difference objective to emulate GAIMS behavior. Optimize with SAGA-optimize (simulated annealing plus genetic algorithm), supplemented by scipy optimization methods, running many stochastic repeats (e.g. --repetition=100) with --split and --multiprocess to sample the search landscape. Analyze the optimized results with 'analyze optimizations' and rank the candidate models with 'analyze rank --rankCriteria=AICc'. Visualize fitted moiety curves with numpy and matplotlib, and compare fitted parameter values and rank order against reference outputs. Because the algorithm is stochastic and the objective function strongly affects model selection (absolute-difference fitting can overfit and defeat AICc ranking), always run repetitions and compare rank/parameter outputs rather than trusting a single optimization run.

## Related tools

- **SAGA-optimize** (Core optimizer solving the boundary-value inverse problem through combined simulated annealing and genetic algorithm)
- **moiety_modeling** (Package providing moiety model representation, optimization entry points, analysis, ranking, and plotting commands) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Supplementary optimization methods applied after SAGA to refine fitted parameters) — https://github.com/scipy/scipy
- **pickle** (Saving Python optimization objects in a JSON serializable form and outputting to a file)
- **matplotlib** (Visualization of fitted moiety results via 'plot moiety') — https://github.com/matplotlib/matplotlib

## Examples

```
python3 -m moiety_modeling modeling --models=<model_jsonfile> --datasets=<dataset_jsonfile> --optimizations=<optimizationSetting_json> --repetition=100 --split --multiprocess
```

## Evaluation signals

- The reproduced model rank order by AICc matches the rank comparison reported in Additional file 9.
- Optimized parameter values match the parameter table in Additional file 11.
- Fitted isotopologue fraction curves match the parameter graphs in Additional file 10.
- When run with the absolute-difference objective, results are reasonably comparable (though not identical) to GAIMS output for the UDP-GlcNAc models.
- Multiple optimization repetitions (e.g. --repetition=100) converge to consistent fits rather than a single stochastic outlier.

## Limitations

- Results are not identical to GAIMS when using the matched absolute-difference objective, due to implementation and stochastic differences between the tools.
- With the absolute-difference objective function, neither moiety_modeling nor GAIMS could select the expert-derived model by AICc because of overfitting with that objective.
- As a combined simulated annealing and genetic algorithm, SAGA-optimize is stochastic, so single runs may vary; many repetitions are needed for reliable model ranking.
- Future work is needed on data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection.

## Evidence

- [abstract] SAGA-optimize solves a boundary-value inverse problem through a combined simulated annealing and genetic algorithm: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [methods] AICc ranking command: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [readme] Package scope: "'moiety_modeling' package provides facilities for moiety model representation, model optimization and model selection."
