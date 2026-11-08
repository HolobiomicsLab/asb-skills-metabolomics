---
name: aicc-model-ranking
description: Use when multiple plausible moiety/deconvolution models exist for MS-measured
  isotopologue timecourses and you must objectively select the model, e.g. ranking
  41 UDP-GlcNAc models fitted with SAGA by AICc.
when_to_use_negative:
- The optimization objective function is an absolute difference matched to GAIMS behavior,
  where overfitting makes AICc ranking unable to recover the correct model
- Only a single plausible deconvolution model exists, so no model selection among
  competitors is required
- Isotopologue data are heavily degraded (e.g. added normal error with sigma = 0.1
  on multi-tracer data, where propagated error reaches roughly nine times the original
  signal), making model selection unreliable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: moiety_modeling
  role: Provides the 'analyze rank --rankCriteria=AICc' command that computes AICc
    scores and ranks fitted moiety models
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Combined simulated annealing and genetic algorithm optimizer that produces
    the fitted parameters for each candidate model prior to ranking
- name: scipy
  role: Supplementary optimization methods used alongside SAGA when fitting candidate
    models before ranking
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes the ranked analysis results to a JSON file
  repo: https://github.com/jsonpickle/jsonpickle
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:cbb9f794892f6499dfcd07331c80b86f9729128384eaf18a162f8e7828ffe535
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - jsonpickle
  - matplotlib
  - moiety_modeling
  - numpy
  - pickle
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# aicc-model-ranking

## Summary

Ranks candidate moiety models fitted to MS isotopologue timecourses using the small-sample-corrected Akaike information criterion (AICc), enabling selection of the optimal isotopologue deconvolution model from a set of plausible alternatives.

## When to use

Use when multiple plausible moiety/deconvolution models exist for MS-measured isotopologue timecourses and you must objectively select the best model, e.g. ranking 41 UDP-GlcNAc models fitted with SAGA by AICc.

## When NOT to use

- The optimization objective function is an absolute difference matched to GAIMS behavior, where overfitting makes AICc ranking unable to recover the correct model
- Only a single plausible deconvolution model exists, so no model selection among competitors is required
- Isotopologue data are heavily degraded (e.g. added normal error with sigma = 0.1 on multi-tracer data, where propagated error reaches roughly nine times the original signal), making model selection unreliable

## Inputs

- analysisPaths_txtfile listing consolidated optimization result files from 'moiety_modeling analyze optimizations'
- fitted moiety model parameters from SAGA optimization (serialized via pickle/jsonpickle)
- candidate moiety model definitions (models JSON, e.g. expert-derived 6_G1R1A1U3 plus 40 perturbation models)
- isotopologue timecourse measurements (datasets JSON, e.g. UDP-GlcNAc 13C isotopologues at 34 h, 48 h, 72 h)

## Outputs

- ranked model-selection JSON file (written via jsonpickle) listing each model's AICc value and rank
- per-model AICc scores identifying the optimal model (e.g. AICc = -229.2918 for 6_G1R1A1U3)

## How to apply

First optimize the parameters of every candidate moiety model against the isotopologue timecourse data by running 'python3 -m moiety_modeling modeling' with SAGA (simulated annealing plus genetic algorithm) from SAGA-optimize, e.g. --repetition=100, --split, --multiprocess and --energyFunction=logDifference, consolidating fits via 'analyze optimizations'. Then rank all fitted models with 'python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc', which applies the small-sample-corrected Akaike information criterion to penalize over-parameterized competitors; the ranked results are written to a JSON file serialized with jsonpickle. As a decision point, verify the top-ranked model's AICc against known reference values (e.g. expert-derived 6_G1R1A1U3 at -229.2918 vs competitor 7_G2R1A1U3_g5 at -208.1346). Note that AICc selection is objective-function dependent: with an absolute-difference objective matching GAIMS, overfitting prevented selection of the correct model, so the logDifference function is preferred. Finally, if the generating model was known (simulated data), check whether it ranks first to assess selection robustness across noise levels such as sigma in {0.001, 0.01, 0.1}.

## Related tools

- **moiety_modeling** (Provides the 'analyze rank --rankCriteria=AICc' command that computes AICc scores and ranks fitted moiety models) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Combined simulated annealing and genetic algorithm optimizer that produces the fitted parameters for each candidate model prior to ranking)
- **scipy** (Supplementary optimization methods used alongside SAGA when fitting candidate models before ranking) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes the ranked analysis results to a JSON file) — https://github.com/jsonpickle/jsonpickle

## Examples

```
python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- For the UDP-GlcNAc 13C isotopologue benchmark, the ranking must place the expert-derived model 6_G1R1A1U3 first with AICc = -229.2918, ahead of 6_G1R1A1U3_r4 (-227.5208) and 7_G2R1A1U3_g5 (-208.1346)
- The rank output JSON must be loadable via jsonpickle and contain AICc criteria values for all candidate models
- For simulated datasets with known generating models, the generating model should rank first by AICc in the noise-free limit
- Rank order should be reproducible across 100 SAGA optimization repetitions and comparable (though not identical) to GAIMS results under the same absolute-difference objective

## Limitations

- AICc ranking with an absolute-difference objective function could not select the expert-derived model because of overfitting, matching GAIMS behavior
- Optimization and ranking results are not identical to GAIMS due to implementation and stochastic differences, only reasonably comparable
- High added-error levels (sigma = 0.1) make multi-tracer simulated datasets practically unusable, with propagated error reaching roughly nine times the original signal on average
- Results depend on stochastic SAGA optimization; the rank command was run with 100 repetitions per model to achieve robustness

## Evidence

- [methods] rank models using AICc criteria: "python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc"
- [readme] package capabilities: "'moiety_modeling' package provides facilities for moiety model representation, model optimization and model selection."
- [intro] model selection motivation: "when multiple models are plausible, development of a robust model selection method is essential for successful isotopologue deconvolution"
- [abstract] SAGA optimizer role: "The SAGA-optimize package, solving a boundary-value inverse problem through a combined simulated annealing and genetic algorithm, was developed for model optimization"
