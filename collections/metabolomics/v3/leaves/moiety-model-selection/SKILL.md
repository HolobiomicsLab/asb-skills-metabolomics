---
name: moiety-model-selection
description: Use when multiple plausible isotopologue deconvolution models exist for
  a metabolite (e.g., UDP-GlcNAc measured by 13C SIRM at multiple timepoints) and
  you must objectively select the model via AICc rather than fit quality alone.
when_to_use_negative:
- Data are from a single well-curated metabolic network suitable for full EMU-based
  13C MFA rather than moiety-level deconvolution
- Only one plausible deconvolution model exists and model selection is unnecessary
  — direct parameter optimization suffices
- Handling only 13C single isotope tracer data with the MAIMS-style workflow that
  does not require model selection
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: moiety_modeling
  role: Provides the CLI for model representation, optimization ('modeling'), consolidation
    ('analyze optimizations'), and AICc ranking ('analyze rank --rankCriteria=AICc')
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Performs the underlying model optimization via a combined simulated annealing
    and genetic algorithm for the boundary-value inverse problem
- name: scipy
  role: Supplies additional optimization methods used alongside SAGA for parameter
    fitting
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the ranked analysis results to a JSON file
  repo: https://github.com/jsonpickle/jsonpickle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:8ca1c01d414cbab72e2cea2053315973c947c125076b5f5e9196f50c49dc3c81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - jsonpickle
  - moiety_modeling
  - scipy
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# moiety-model-selection

## Summary

Selects the optimal moiety model for MS isotopologue deconvolution by fitting a set of candidate models (e.g., an expert-derived model plus perturbations) to 13C isotopologue timecourses with SAGA optimization and ranking them by the small-sample-corrected AICc criterion.

## When to use

Use when multiple plausible isotopologue deconvolution models exist for a metabolite (e.g., UDP-GlcNAc measured by 13C SIRM at multiple timepoints) and you must objectively select the best model via AICc rather than fit quality alone.

## When NOT to use

- Data are from a single well-curated metabolic network suitable for full EMU-based 13C MFA rather than moiety-level deconvolution
- Only one plausible deconvolution model exists and model selection is unnecessary — direct parameter optimization suffices
- Handling only 13C single isotope tracer data with the MAIMS-style workflow that does not require model selection

## Inputs

- models JSON file (expert-derived moiety model plus perturbation models, e.g., 6_G1R1A1U3 and ~40 variants)
- datasets JSON file (13C isotopologue timecourse measurements at 34 h, 48 h, 72 h)
- optimization settings JSON file (energyFunction, repetitions, splitting, multiprocessing)
- optimizationPaths_txtfile listing optimization output paths

## Outputs

- Optimization result files from SAGA fitting (one per model per repetition)
- Consolidated analysis files from 'analyze optimizations'
- Ranked analysis JSON written via jsonpickle with AICc values per model
- Ranking placing the expert-derived model first (AICc ≈ −229.2918 in the UDP-GlcNAc case)

## How to apply

Assemble a models JSON file containing the expert-derived model (e.g., 6_G1R1A1U3) plus perturbation competitors, a datasets JSON with the 13C isotopologue timecourse measurements (e.g., 34 h, 48 h, 72 h), and an optimization settings JSON. Run 'python3 -m moiety_modeling modeling' with --repetition=100, --split, --multiprocess, and --energyFunction=logDifference, which delegates the boundary-value inverse problem to the SAGA (simulated annealing plus genetic algorithm) optimizer. Consolidate fits with 'analyze optimizations --a <optimizationPaths_txtfile>', then rank with 'analyze rank <analysisPaths_txtfile> --rankCriteria=AICc' using the small-sample-corrected Akaike information criterion. The correct model is the one with the lowest AICc, not merely the best objective score, which guards against overfitting by parameter-rich perturbation models.

## Related tools

- **moiety_modeling** (Provides the CLI for model representation, optimization ('modeling'), consolidation ('analyze optimizations'), and AICc ranking ('analyze rank --rankCriteria=AICc')) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Performs the underlying model optimization via a combined simulated annealing and genetic algorithm for the boundary-value inverse problem)
- **scipy** (Supplies additional optimization methods used alongside SAGA for parameter fitting) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the ranked analysis results to a JSON file) — https://github.com/jsonpickle/jsonpickle

## Examples

```
python3 -m moiety_modeling modeling --repetition=100 --split --multiprocess --energyFunction=logDifference && python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- The expert-derived model 6_G1R1A1U3 ranks first with the lowest AICc ≈ −229.2918 across all candidate models
- Known competitor models match reported values, e.g., 6_G1R1A1U3_r4 at AICc ≈ −227.5208 and 7_G2R1A1U3_g5 at AICc ≈ −208.1346
- Each model is optimized over ~100 SAGA repetitions with the logDifference energy function, and results are reproducible across repetitions
- The ranked output JSON is loadable and contains per-model AICc values written via jsonpickle

## Limitations

- Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection
- AICc ranking assumes the candidate model set includes the true model; overly restricted perturbation sets may exclude the optimal structure
- SAGA optimization is stochastic; ranking reliability depends on sufficient repetitions (100 used in the UDP-GlcNAc demonstration)

## Evidence

- [results] Using SAGA optimization with a log difference objective function (100 repetitions per model), the expert-derived moiety model was successfully selected among all 40 moiety models by AICc: [evidence span withheld — anchor status: not_found]
- [methods] workflow ranking step: [evidence span withheld — anchor status: not_found]
- [abstract] optimizer role: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
- [discussion] data-quality caveat: "Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection."
- [readme] package purpose: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
