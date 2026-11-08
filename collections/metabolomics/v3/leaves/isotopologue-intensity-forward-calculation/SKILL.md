---
name: isotopologue-intensity-forward-calculation
description: Use when a measured MS isotopologue profile of a multi-moiety metabolite
  (e.g. single-tracer 13C UDP-GlcNAc timecourses or multi-tracer 13C/18O data) must
  be reconstructed from moiety state fractional abundances, i.e. computing I_x,calc
  per isotopologue mass shift from a moiety model.
when_to_use_negative:
- The data are raw MS peaks rather than isotopologue profiles of a metabolite decomposable
  into moieties — the framework assumes metabolite decomposition into moiety units
  has already been defined
- Only a single-isotope tracer with no model-structure ambiguity exists and an existing
  tool is acceptable — note the precedent that 'the MAIMS software handles only 13C
  single isotope tracer data and does not address model selection', whereas this skill
  exists to support single AND multiple-labeled SIRM datasets with explicit model
  selection
edam_operation: http://edamontology.org/operation_2945
tools:
- name: moiety_modeling
  role: Reference implementation of the moiety model representation and the Eq. 1
    forward calculation; its CLI takes --models and --datasets JSON inputs and its
    output serves as the validation reference
  repo: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: numpy
  role: Enumerating moiety state combinations and computing the product-summation
    over IC_x components for each isotopologue mass shift
- name: scipy
  role: Provides the optimization methods used by moiety_modeling to fit the moiety
    state parameters that feed the forward calculation
  repo: https://github.com/scipy/scipy
- name: jsonpickle
  role: Serializes optimized moiety model parameter objects to JSON-serializable form
    for input/output
  repo: https://github.com/jsonpickle/jsonpickle
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:119fa496c5980d812c3d95a00c8e51834e8407b5ce01711abe0d08b71cfa289e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/moiety_modeling
  tools:
  - jsonpickle
  - moiety_modeling
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# isotopologue-intensity-forward-calculation

## Summary

Reconstructs the forward isotopologue-intensity calculation (Eq. 1) of the moiety modeling framework: the calculated relative intensity I_x,calc for isotope content x is the sum over all isotopologue components ic_v in IC_x of the products of moiety state fractional abundances across all moieties. This is the core forward model that moiety_modeling fits against measured MS isotopologue profiles of single- and multi-tracer SIRM data.

## When to use

Use when a measured MS isotopologue profile of a multi-moiety metabolite (e.g. single-tracer 13C UDP-GlcNAc timecourses or multi-tracer 13C/18O data) must be reconstructed from moiety state fractional abundances, i.e. computing I_x,calc per isotopologue mass shift from a moiety model.

## When NOT to use

- The data are raw MS peaks rather than isotopologue profiles of a metabolite decomposable into moieties — the framework assumes metabolite decomposition into moiety units has already been defined
- Only a single-isotope tracer with no model-structure ambiguity exists and an existing tool is acceptable — note the precedent that 'the MAIMS software handles only 13C single isotope tracer data and does not address model selection', whereas this skill exists to support single AND multiple-labeled SIRM datasets with explicit model selection

## Inputs

- moiety model definitions in a JSON file (--models=<model_jsonfile>), including moiety groups and moiety state parameters
- measured isotopologue profiles in a dataset JSON file (--datasets=<dataset_jsonfile>)
- optimized moiety model parameter objects (pickle, JSON-serializable via jsonpickle) from moiety_modeling optimization runs

## Outputs

- calculated_isotopologue_intensities.csv with normalized calculated relative isotopologue intensities (I_x,calc) for every isotopologue of both the single-tracer 13C and multi-tracer 13C/18O models
- calculated isotopologue intensity profiles comparable against measured profiles and against the published moiety_modeling reference output

## How to apply

Load the moiety model definitions from the model JSON file (--models=<model_jsonfile>) and the measured isotopologue profiles from the dataset JSON file (--datasets=<dataset_jsonfile>). For each moiety group, enumerate all moiety state combinations and compute each isotopologue component ic_v as the product of the moiety states across all moieties (moiety_state_{j,v_j}); IC_x is the set of components whose isotope content equals x. Sum the IC_x components per isotopologue mass shift to obtain I_x,calc, extending the same product-summation scheme to the multi-tracer 13C/18O model where carbon and oxygen moiety states combine. Normalize the calculated profiles to relative intensities and write both models' results to calculated_isotopologue_intensities.csv. The rationale is that this Eq. 1 forward map is exactly what moiety_modeling's parameter optimization (SAGA-optimize plus scipy optimizers) evaluates, so reproducing it from scratch validates the deconvolution semantics for both single and multiple isotope tracers.

## Related tools

- **moiety_modeling** (Reference implementation of the moiety model representation and the Eq. 1 forward calculation; its CLI takes --models and --datasets JSON inputs and its output serves as the validation reference) — https://github.com/MoseleyBioinformaticsLab/moiety_modeling
- **numpy** (Enumerating moiety state combinations and computing the product-summation over IC_x components for each isotopologue mass shift)
- **scipy** (Provides the optimization methods used by moiety_modeling to fit the moiety state parameters that feed the forward calculation) — https://github.com/scipy/scipy
- **jsonpickle** (Serializes optimized moiety model parameter objects to JSON-serializable form for input/output) — https://github.com/jsonpickle/jsonpickle

## Examples

```
python3 -m moiety_modeling --models=<model_jsonfile> --datasets=<dataset_jsonfile>
```

## Evaluation signals

- From-scratch calculated intensities match a reference profile generated with the published moiety_modeling package on identical model/dataset inputs (output_matches_reference)
- Correct field presence for every isotopologue of both models in calculated_isotopologue_intensities.csv
- For each isotope content x, I_x,calc equals the sum of all ic_v components in IC_x, each computed as the product of moiety_state_{j,v_j} across all moieties (Eq. 1 invariant)
- Calculated profiles are normalized to relative intensities comparable to the measured isotopologue profiles
- The same forward calculation reproduces both the single-tracer 13C and the multi-tracer 13C/18O simulated datasets from the fitted moiety state values

## Limitations

- The forward calculation depends on having the correct moiety model — 'The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux.'
- Multi-isotope model optimization and selection are not yet fully validated: 'Future work will explore the data quality requirements of model selection and validation of multiple isotope tracing model optimization and selection.'
- Data quality requirements for reliable model selection are an open question flagged in the article's discussion

## Evidence

- [readme] The moiety_modeling package provides facilities for moiety model representation, model optimization and model selection.: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
- [methods] Using moiety_modeling to optimize parameters of moiety model.: "Using moiety_modeling to optimize parameters of moiety model."
- [intro] we have developed a new moiety modeling framework for deconvoluting MS isotopologue profiles for both single and multiple-labeled SIRM MS datasets: "we have developed a new moiety modeling framework for deconvoluting MS isotopologue profiles for both single and multiple-labeled SIRM MS datasets"
- [discussion] The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux.: "The selection of correct moiety models is required for generating deconvolution results that can be accurately interpreted in terms of relative metabolic flux."
