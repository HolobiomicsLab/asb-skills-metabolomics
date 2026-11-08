---
name: moiety-state-fractional-abundance-mapping
description: Use when measured MS isotopologue profiles of a metabolite (e.g., UDP-GlcNAc
  timecourses) must be reproduced or deconvoluted from moiety-state fractional abundances,
  or when simulating single-tracer 13C or multi-tracer 13C/18O isotopologue data from
  a moiety model JSON.
when_to_use_negative:
- The data are raw MS spectra without an assigned moiety model; the framework requires
  a plausible moiety model and needs model selection (AICc ranking) before the forward
  calculation is interpretable
- The analysis is quantitative metabolic flux analysis over curated metabolic networks
  — the article notes EMU-style methods serve that purpose, whereas moiety modeling
  targets isotopologue profile deconvolution
- Input is 13C single-tracer data already handled by MAIMS without needing model selection;
  this skill's value is mainly multi-tracer and model-selection settings
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: moiety_modeling
  role: provides moiety model representation and the reference implementation of the
    Eq. 1 forward intensity calculation used for validation
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: supplies optimized moiety-state fractional abundance parameters via combined
    simulated annealing and genetic algorithm before the forward calculation
- name: scipy
  role: additional optimization methods used to fit moiety-state parameters whose
    products feed Eq. 1
  repo: https://github.com/scipy/scipy
- name: numpy
  role: numerical evaluation of the product-summation over isotopologue components
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:fbab075d26a3a54115c93f312b3cd1374026d374680da6041fe640a4af20d5ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - SAGA-optimize
  - moiety_modeling
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# moiety-state-fractional-abundance-mapping

## Summary

Reconstructs the forward isotopologue-intensity calculation (Eq. 1) of the moiety modeling framework, where the calculated relative isotopologue intensity I(x,calc) is the sum over isotopologue components whose isotope content equals x, each component being the product of moiety-state fractional abundances across all moieties. This underpins isotopologue deconvolution for single- and multi-tracer (13C, 18O) SIRM MS data.

## When to use

Use when measured MS isotopologue profiles of a metabolite (e.g., UDP-GlcNAc timecourses) must be reproduced or deconvoluted from moiety-state fractional abundances, or when simulating single-tracer 13C or multi-tracer 13C/18O isotopologue data from a moiety model JSON.

## When NOT to use

- The data are raw MS spectra without an assigned moiety model; the framework requires a plausible moiety model and needs model selection (AICc ranking) before the forward calculation is interpretable
- The analysis is quantitative metabolic flux analysis over curated metabolic networks — the article notes EMU-style methods serve that purpose, whereas moiety modeling targets isotopologue profile deconvolution
- Input is 13C single-tracer data already handled by MAIMS without needing model selection; this skill's value is mainly multi-tracer and model-selection settings

## Inputs

- moiety model definitions JSON file (--models=<model_jsonfile>) specifying moiety groups and moiety states
- measured isotopologue profiles dataset JSON file (--datasets=<dataset_jsonfile>)
- optimized moiety-state fractional abundance parameters (e.g., from SAGA-optimize or scipy optimization)

## Outputs

- calculated relative isotopologue intensity profile I(x,calc) per mass shift x
- calculated_isotopologue_intensities.csv with normalized calculated intensities for every isotopologue of both single-tracer and multi-tracer models

## How to apply

Load the moiety model definitions from the model JSON file (--models=<model_jsonfile>) and the measured isotopologue profiles from the dataset JSON file (--datasets=<dataset_jsonfile>). For each isotopologue mass shift x, identify the set IC_x of isotopologue components ic_v whose isotope content equals x, and compute each component as the product of the moiety-state fractional abundances (moiety_state_{j,v_j}) across all moieties; sum these products to obtain I(x,calc) per Eq. 1. For the multi-tracer 13C/18O model, combine carbon and oxygen moiety states under the same product-summation scheme. Normalize the calculated profiles and write them out (e.g., calculated_isotopologue_intensities.csv); the same Eq. 1 scheme generates both single-tracer and multi-tracer simulated datasets. Validate by generating a reference profile with the published moiety_modeling package on identical model/dataset inputs and checking the from-scratch intensities match.

## Related tools

- **moiety_modeling** (provides moiety model representation and the reference implementation of the Eq. 1 forward intensity calculation used for validation) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (supplies optimized moiety-state fractional abundance parameters via combined simulated annealing and genetic algorithm before the forward calculation)
- **scipy** (additional optimization methods used to fit moiety-state parameters whose products feed Eq. 1) — https://github.com/scipy/scipy
- **numpy** (numerical evaluation of the product-summation over isotopologue components)

## Examples

```
python3 -m moiety_modeling analyze rank <analysisPaths_txtfile> --rankCriteria=AICc
```

## Evaluation signals

- output_matches_reference: from-scratch calculated intensities match a reference profile generated by the published moiety_modeling package on identical --models and --datasets inputs
- field presence: calculated_isotopologue_intensities.csv contains entries for every isotopologue of both the single-tracer 13C model and the multi-tracer 13C/18O model
- for each isotopologue x, I(x,calc) equals the sum of products over all ic_v in IC_x with isotope content equal to x (Eq. 1 invariant)
- calculated profiles are normalized (relative intensities sum consistently across isotopologues)
- the same Eq. 1 code path reproduces both the single-tracer and multi-tracer simulated datasets without model-specific special-casing

## Limitations

- moiety_modeling must be cited via the GitHub repository until the manuscript version; it runs under Python 3.6+
- The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux — a wrong model invalidates the mapped intensities
- Future work still needs to explore data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection
- The framework was validated on 13C isotopologue timecourses of UDP-GlcNAc in LnCaP-LN3 cells; other metabolites/tracers are untested in this context

## Evidence

- [readme] The moiety_modeling package provides facilities for moiety model representation, model optimization and model selection.: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
- [abstract] we were able to confirm its robust performance in isotopologue deconvolution and moiety model selection: "we were able to confirm its robust performance in isotopologue deconvolution and moiety model selection"
- [methods] Using moiety_modeling to optimize parameters of moiety model.: "Using moiety_modeling to optimize parameters of moiety model."
- [abstract] SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool: "SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool"
- [abstract] Both packages are freely available on GitHub and via the Python Package Index: "Both packages are freely available on GitHub and via the Python Package Index"
