---
name: isotope-flux-model-perturbation-design
description: Use when you have MS-measured isotopologue timecourses (e.g. 13C UDP-GlcNAc
  at 34 h, 48 h, 72 h), a plausible expert-derived moiety model, and need to test
  whether that model can be selected from generated perturbation competitors by AICc.
when_to_use_negative:
- When the metabolite is a single unambiguous isotopologue profile with no competing
  plausible moiety models — perturbation design and AICc ranking add no information
- When you need quantitative metabolic flux from a curated metabolic network (EMU-style
  13C MFA) rather than deconvolution and model selection of moiety abundances
- When data are only a single timepoint with low isotopologue counts — the article
  notes data quality requirements of model selection remain unexplored
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: moiety_modeling
  role: CLI package for moiety model representation, parameter optimization, analysis,
    and AICc ranking of the perturbation model set
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Solves the boundary-value inverse problem of fitting each model via combined
    simulated annealing and genetic algorithm with the logDifference energy function
- name: scipy
  role: Provides additional optimization methods used alongside SAGA for parameter
    fitting
- name: jsonpickle
  role: Serializes the ranked analysis results to JSON files
- name: matplotlib
  role: Visualizes optimized moiety results (plot moiety command)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:a8d343ba0ac9b966ef59fc95be25477031e49a893991e8c22e1e9b40fc348853
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# isotope-flux-model-perturbation-design

## Summary

Design and rank perturbation variants of an expert-derived moiety model against mass spectrometry isotopologue timecourses, using SAGA (simulated annealing + genetic algorithm) optimization with a log difference energy function and small-sample-corrected AICc model selection to recover the correct model from a candidate set.

## When to use

Use when you have MS-measured isotopologue timecourses (e.g. 13C UDP-GlcNAc at 34 h, 48 h, 72 h), a plausible expert-derived moiety model, and need to test whether that model can be selected from generated perturbation competitors by AICc.

## When NOT to use

- When the metabolite is a single unambiguous isotopologue profile with no competing plausible moiety models — perturbation design and AICc ranking add no information
- When you need quantitative metabolic flux from a curated metabolic network (EMU-style 13C MFA) rather than deconvolution and model selection of moiety abundances
- When data are only a single timepoint with low isotopologue counts — the article notes data quality requirements of model selection remain unexplored

## Inputs

- models JSON file with expert-derived moiety model (e.g. 6_G1R1A1U3) and ~40 perturbation variants
- datasets JSON file with 13C isotopologue timecourse measurements (34 h, 48 h, 72 h; UDP-GlcNAc in LnCaP-LN3 cells)
- optimization settings JSON file
- text file listing optimization output paths (for the analyze step)

## Outputs

- optimization output paths collected in a text file
- consolidated analysis of fitted models (analysisPaths file)
- ranked analysis results JSON (jsonpickle-serialized) with per-model AICc values

## How to apply

Assemble three JSON inputs: a models JSON containing the expert-derived moiety model (e.g. 6_G1R1A1U3) plus systematically generated perturbation models (40 variants such as 6_G1R1A1U3_r4, 7_G2R1A1U3_g5), a datasets JSON with isotopologue measurements at each timepoint, and an optimization settings JSON. Run parameter optimization via 'python3 -m moiety_modeling modeling' with --repetition=100, --split, --multiprocess and --energyFunction=logDifference, delegating the boundary-value inverse problem to SAGA-optimize (combined simulated annealing and genetic algorithm); scipy optimizers are also available. Consolidate fitted results with 'analyze optimizations', then rank all candidates with 'analyze rank --rankCriteria=AICc', the small-sample-corrected Akaiai information criterion, writing ranked results via jsonpickle. Success means the expert-derived model ranks first with the lowest AICc (≈ −229.2918 for 6_G1R1A1U3 vs −227.5208 for 6_G1R1A1U3_r4), demonstrating the framework's robustness.

## Related tools

- **moiety_modeling** (CLI package for moiety model representation, parameter optimization, analysis, and AICc ranking of the perturbation model set) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Solves the boundary-value inverse problem of fitting each model via combined simulated annealing and genetic algorithm with the logDifference energy function)
- **scipy** (Provides additional optimization methods used alongside SAGA for parameter fitting)
- **jsonpickle** (Serializes the ranked analysis results to JSON files)
- **matplotlib** (Visualizes optimized moiety results (plot moiety command))

## Examples

```
python3 -m moiety_modeling modeling --repetition=100 --split --multiprocess --energyFunction=logDifference; python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- Expert-derived model 6_G1R1A1U3 ranks first with lowest AICc ≈ −229.2918 across all 41 candidates
- Known competitor values reproduced: 6_G1R1A1U3_r4 at AICc ≈ −227.5208 and 7_G2R1A1U3_g5 at AICc ≈ −208.1346
- Each model fit uses 100 SAGA repetitions with --energyFunction=logDifference, so repeated runs should stably select the same top model
- Ranked output is a valid jsonpickle-serialized JSON file readable by the analyze/plot subcommands

## Limitations

- Future work must explore the data quality requirements of model selection and validation of multiple-isotope tracing model optimization and selection
- Selection of the correct moiety model is required for deconvolution results to be interpretable in terms of relative metabolic flux — a wrongly ranked model invalidates downstream interpretation
- Existing MAIMS-style approaches handle only 13C single-tracer data and no model selection; multi-tracer isotopologue profiles complicate the analysis

## Evidence

- [abstract] SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool: "SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool"
- [methods] rank models using AICc criteria: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [readme] moiety_modeling capabilities: "moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
- [abstract] robustness finding: "we were able to confirm its robust performance in isotopologue deconvolution and moiety model selection"
