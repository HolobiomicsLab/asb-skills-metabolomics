---
name: isotopologue-timecourse-parameter-optimization
description: Use when you need to solve the boundary-value inverse problem of fitting
  moiety model parameters to MS-measured isotopologue timecourse data, and/or to select
  among multiple plausible moiety models via AICc.
when_to_use_negative:
- The data are 13C single-tracer profiles already handled by MAIMS and no model selection
  among competing moiety models is needed (MAIMS handles 13C single isotope tracer
  data but does not address model selection).
- The goal is full metabolic flux analysis over a well-curated metabolic network (EMU-style
  13C MFA), rather than deconvoluting isotopologue profiles into moiety abundances.
- The data are NMR-based isotopomer measurements or unlabeled abundance data with
  no isotope tracer timecourse to fit.
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0203
tools:
- name: moiety_modeling
  role: Command-line package providing moiety model representation, parameter optimization,
    analysis, AICc ranking, and plotting of isotopologue timecourse fits
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Optimizer solving the boundary-value inverse problem via combined simulated
    annealing and genetic algorithm under the log difference energy function
- name: scipy
  role: Provides additional optimization methods used alongside SAGA for model parameter
    fitting
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the ranked analysis results (Python objects) to JSON files
  repo: https://github.com/jsonpickle/jsonpickle
- name: matplotlib
  role: Used by the plot subcommand to visualize optimized moiety isotopologue profiles
  repo: https://github.com/matplotlib/matplotlib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:ad9cc26b482c8f38c434090c05668a5c2b0ed03c5e22b649956a5f8491943072
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - jsonpickle
  - matplotlib
  - moiety_modeling
  - scipy
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# isotopologue-timecourse-parameter-optimization

## Summary

Fit moiety model parameters to mass spectrometry-measured 13C isotopologue timecourses (e.g., UDP-GlcNAc at 34 h, 48 h, 72 h) using the SAGA simulated annealing plus genetic algorithm optimizer driven by a log difference objective function, then rank fitted models by AICc for model selection.

## When to use

Use when you need to solve the boundary-value inverse problem of fitting moiety model parameters to MS-measured isotopologue timecourse data, and/or to select among multiple plausible moiety models via AICc.

## When NOT to use

- The data are 13C single-tracer profiles already handled by MAIMS and no model selection among competing moiety models is needed (MAIMS handles 13C single isotope tracer data but does not address model selection).
- The goal is full metabolic flux analysis over a well-curated metabolic network (EMU-style 13C MFA), rather than deconvoluting isotopologue profiles into moiety abundances.
- The data are NMR-based isotopomer measurements or unlabeled abundance data with no isotope tracer timecourse to fit.

## Inputs

- models JSON file listing candidate moiety models (e.g., expert-derived 6_G1R1A1U3 and perturbation models)
- datasets JSON file with 13C isotopologue timecourse measurements (e.g., UDP-GlcNAc at 34 h, 48 h, 72 h)
- optimization settings JSON file (repetitions, energy function, split/multiprocess flags)
- text file of optimization output paths (--a argument for analyze optimizations)

## Outputs

- optimization result files (one per model per repetition) with fitted moiety abundances and objective scores
- analysis paths text file feeding the ranking step
- ranked model-selection JSON report saved via jsonpickle with AICc values per model
- plots of optimized moiety isotopologue profiles versus measured data

## How to apply

Assemble three inputs: a models JSON file containing the candidate moiety models (e.g., expert-derived 6_G1R1A1U3 plus perturbation variants), a datasets JSON file with the 13C isotopologue timecourse measurements, and an optimization settings JSON file. Run parameter optimization with 'python3 -m moiety_modeling modeling' using --repetition=100 (multiple repetitions because SAGA is stochastic), --split, --multiprocess, and --energyFunction=logDifference, which delegates to the SAGA-optimize package combining simulated annealing and a genetic algorithm. Consolidate fitted results with 'python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>'. Rank the fitted models with 'python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc' using the small-sample-corrected Akaike information criterion, and optionally visualize fitted versus measured isotopologue profiles with 'python3 -m moiety_modeling plot moiety <analysisResults_file>'. Results are serialized to JSON via jsonpickle for downstream reuse.

## Related tools

- **moiety_modeling** (Command-line package providing moiety model representation, parameter optimization, analysis, AICc ranking, and plotting of isotopologue timecourse fits) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Optimizer solving the boundary-value inverse problem via combined simulated annealing and genetic algorithm under the log difference energy function)
- **scipy** (Provides additional optimization methods used alongside SAGA for model parameter fitting) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the ranked analysis results (Python objects) to JSON files) — https://github.com/jsonpickle/jsonpickle
- **matplotlib** (Used by the plot subcommand to visualize optimized moiety isotopologue profiles) — https://github.com/matplotlib/matplotlib

## Examples

```
python3 -m moiety_modeling modeling --models models.json --datasets datasets.json --settings settings.json --repetition=100 --split --multiprocess --energyFunction=logDifference
```

## Evaluation signals

- With 100 SAGA repetitions per model, the ranking by --rankCriteria=AICc should place the true (expert-derived) model 6_G1R1A1U3 first with AICc ≈ −229.2918, as reported for the UDP-GlcNAc timecourse benchmark.
- Known competitor models should reproduce reported scores, e.g., 6_G1R1A1U3_r4 at AICc ≈ −227.5208 and 7_G2R1A1U3_g5 at AICc ≈ −208.1346.
- The analyze optimizations step must consume the optimization paths text file and produce analysis result files loadable by the rank step without errors.
- Ranked results should be written as a JSON file serialized with jsonpickle and reloadable as Python objects.
- Fitted versus measured isotopologue profiles across the 34 h, 48 h, and 72 h timepoints should visually agree when plotted with 'plot moiety'.

## Limitations

- SAGA is a stochastic optimizer, so multiple repetitions (the paper used 100) are required per model to obtain stable fits, and results vary between runs.
- Data quality requirements for model selection and validation of multiple-isotope-tracing model optimization and selection remain unexplored future work.
- Accurate interpretation in terms of relative metabolic flux depends on selecting the correct moiety model; a wrong model yields misleading deconvolution.

## Evidence

- [methods] SAGA optimization with log difference objective over 100 repetitions; AICc ranking selects expert model: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [abstract] SAGA delegates boundary-value inverse problem solving: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [abstract] Framework validated on UDP-GlcNAc 13C timecourse in LnCaP-LN3 cells: "we were able to confirm its robust performance in isotopologue deconvolution and moiety model selection"
- [readme] Packages available on GitHub and PyPI: "'moiety_modeling' package provides facilities for moiety model representation, model optimization and model selection."
- [discussion] Model selection required for accurate flux interpretation: "The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux."
