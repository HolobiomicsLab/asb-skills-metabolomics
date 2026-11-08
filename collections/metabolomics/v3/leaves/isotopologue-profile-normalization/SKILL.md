---
name: isotopologue-profile-normalization
description: Use when mass spectrometry isotopologue profiles (e.g. 13C UDP-GlcNAc
  timecourses) must be reconstructed from moiety state fractional abundances, or when
  validating deconvolution outputs against measured isotopologue distributions.
when_to_use_negative:
- The metabolite is treated with quantitative flux approaches like the EMU framework,
  which rely on well-curated metabolic networks rather than moiety decomposition
- Only a single 13C tracer dataset with no model-selection need is present and the
  existing MAIMS software already suffices (MAIMS handles only 13C single tracer data
  and does not address model selection)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: moiety_modeling
  role: Package providing moiety model representation, optimization and selection;
    its Eq. 1 forward calculation is reconstructed here and used to generate the simulated
    single-tracer and multi-tracer datasets
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: SAGA-optimize
  role: Combined simulated annealing and genetic algorithm optimizer for the boundary-value
    inverse problem whose forward function is the Eq. 1 intensity calculation
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: scipy
  role: Additional optimization methods used to fit the forward isotopologue-intensity
    model to measured profiles
  repo: github.com/scipy/scipy
- name: numpy
  role: Numerical computation of products over moiety states and sums over IC_x components
- name: jsonpickle
  role: Used by moiety_modeling for saving Python objects (optimized models/results)
    in a JSON serializable form
  repo: github.com/jsonpickle/jsonpickle
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:f33e0cf68b3e811642cd39b83a91b682e53cacd87e5c843f1975e6a59eb785c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAGA-optimize
  - jsonpickle
  - moiety_modeling
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# isotopologue-profile-normalization

## Summary

Reconstructs the forward isotopologue-intensity calculation (Eq. 1) of the moiety modeling framework, computing relative isotopologue intensities Ix,calc as the sum over isotopologue components IC_x, each a product of moiety state fractional abundances, for single-tracer 13C and multi-tracer 13C/18O models.

## When to use

Use when mass spectrometry isotopologue profiles (e.g. 13C UDP-GlcNAc timecourses) must be reconstructed from moiety state fractional abundances, or when validating deconvolution outputs against measured isotopologue distributions.

## When NOT to use

- The metabolite is treated with quantitative flux approaches like the EMU framework, which rely on well-curated metabolic networks rather than moiety decomposition
- Only a single 13C tracer dataset with no model-selection need is present and the existing MAIMS software already suffices (MAIMS handles only 13C single tracer data and does not address model selection)

## Inputs

- moiety model JSON file (--models=<model_jsonfile>) defining moiety groups and moiety states
- measured isotopologue profiles dataset JSON file (--datasets=<dataset_jsonfile>)
- optimization results files (pickle/jsonpickle objects) from moiety_modeling

## Outputs

- calculated_isotopologue_intensities.csv with normalized relative isotopologue intensities for every isotopologue of both the single-tracer 13C and multi-tracer 13C/18O models
- analysis/ranking outputs (e.g. AICc criteria) when feeding optimized models forward

## How to apply

Load the moiety model definitions from the model JSON file (--models=<model_jsonfile>) and the measured isotopologue profiles from the dataset JSON file (--datasets=<dataset_jsonfile>), both named as command-line inputs in the moiety_modeling quickstart. For a single-tracer 13C model, enumerate all moiety state combinations per moiety group and compute each isotopologue component ic_v as the product of the constituent moiety state fractional abundances (moiety_state_{j,v_j}); components whose isotope content equals x form the set IC_x, and the calculated relative intensity Ix,calc is the sum over that set. Extend the same product-summation scheme to the multi-tracer 13C/18O model, where carbon and oxygen moiety states combine multiplicatively. Normalize the calculated profiles, write them to calculated_isotopologue_intensities.csv, and check field presence for every isotopologue of both models. The rationale is that this Eq. 1 forward model is exactly what the moiety_modeling package (with SAGA-optimize and scipy optimizers) optimizes against measured MS isotopologue data.

## Related tools

- **moiety_modeling** (Package providing moiety model representation, optimization and selection; its Eq. 1 forward calculation is reconstructed here and used to generate the simulated single-tracer and multi-tracer datasets) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **SAGA-optimize** (Combined simulated annealing and genetic algorithm optimizer for the boundary-value inverse problem whose forward function is the Eq. 1 intensity calculation) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **scipy** (Additional optimization methods used to fit the forward isotopologue-intensity model to measured profiles) — github.com/scipy/scipy
- **numpy** (Numerical computation of products over moiety states and sums over IC_x components)
- **jsonpickle** (Used by moiety_modeling for saving Python objects (optimized models/results) in a JSON serializable form) — github.com/jsonpickle/jsonpickle

## Examples

```
python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>
```

## Evaluation signals

- Generate a reference profile with the published moiety_modeling package on identical model/dataset inputs and verify the from-scratch calculated intensities match the reference (output_matches_reference)
- Correct field presence for every isotopologue of both the single-tracer 13C and multi-tracer 13C/18O models in calculated_isotopologue_intensities.csv
- Normalized calculated relative intensities per isotopologue profile sum to 1 (within numerical tolerance)
- For the single-tracer model, Ix,calc for each mass shift equals the sum of product terms over IC_x; perturbing any moiety state fraction must change only the isotopologues containing that moiety
- The same Eq. 1 calculation reproduces the published single-tracer and multi-tracer simulated datasets when applied with the published moiety state values

## Limitations

- Selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux; a wrong model makes Eq. 1 outputs uninterpretable
- Future work must address data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection
- Simultaneous use of multiple stable isotopes in SIRM experiments provides more data but complicates the isotopologue profile analysis

## Evidence

- [intro] we have developed a new moiety modeling framework for deconvoluting MS isotopologue profiles for both single and multiple-labeled SIRM MS datasets: "we have developed a new moiety modeling framework for deconvoluting MS isotopologue profiles for both single and multiple-labeled SIRM MS datasets"
- [abstract] SAGA-optimize is useful for solving boundary-value inverse problems: "SAGA-optimize is a useful Python package for solving boundary-value inverse problems, and the moiety_modeling package is an easy-to-use tool"
- [readme] moiety_modeling package provides facilities for moiety model representation, model optimization and model selection.: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
- [methods] python3 -m moiety_modeling analyze optimizations: "python3 -m moiety_modeling analyze optimizations --a <optimizationPaths_txtfile>"
- [discussion] The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux.: "The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux."
