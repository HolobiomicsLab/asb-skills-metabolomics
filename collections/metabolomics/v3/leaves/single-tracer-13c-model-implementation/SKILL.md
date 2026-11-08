---
name: single-tracer-13c-model-implementation
description: Use when reconstructing or replicating how the moiety_modeling package
  computes calculated relative isotopologue intensities (Ix,calc) from moiety state
  fractional abundances for a 13C single-tracer isotopologue profile.
when_to_use_negative:
- The dataset is a multi-tracer 13C/18O experiment where carbon and oxygen moiety
  states must combine — use the multi-tracer extension of the same Eq. 1 product-summation
  scheme instead
- The goal is metabolic flux analysis over a curated metabolic network (EMU-framework
  style) rather than deconvolution of a single metabolite's MS isotopologue profile
- You only need to run the published package's optimization/ranking (analyze optimizations,
  analyze rank --rankCriteria=AICc) rather than re-implement its forward calculation
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: moiety_modeling
  role: provides the moiety model representation and the reference forward calculation
    of Ix,calc; its model/dataset JSON inputs and quickstart CLI define the interface
    for this skill
  repo: github.com/MoseleyBioinformaticsLab/moiety_modeling
- name: numpy
  role: used to enumerate moiety state combinations and compute the products of moiety
    state fractional abundances and their sums for IC_x
- name: scipy
  role: Python library supplying additional optimization methods used with the moiety
    model in the framework
  repo: github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3096-7
    title: Moiety modeling framework for deriving moiety abundances from mass spectrometry
      measured isotopologues
  source_content_hash: sha256:b6d13b09c265195b537e83b1e687bbd906bd994536a8bf8d947134dd6a83d57c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - moiety_modeling
  - numpy
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# single-tracer-13c-model-implementation

## Summary

Implements the forward isotopologue-intensity calculation (Eq. 1) of the moiety modeling framework: for a single-tracer 13C model, each isotopologue's calculated relative intensity Ix,calc is the sum over isotopologue components whose isotope content equals x, each component being the product of moiety state fractional abundances across all moieties.

## When to use

Use when reconstructing or replicating how the moiety_modeling package computes calculated relative isotopologue intensities (Ix,calc) from moiety state fractional abundances for a 13C single-tracer isotopologue profile.

## When NOT to use

- The dataset is a multi-tracer 13C/18O experiment where carbon and oxygen moiety states must combine — use the multi-tracer extension of the same Eq. 1 product-summation scheme instead
- The goal is metabolic flux analysis over a curated metabolic network (EMU-framework style) rather than deconvolution of a single metabolite's MS isotopologue profile
- You only need to run the published package's optimization/ranking (analyze optimizations, analyze rank --rankCriteria=AICc) rather than re-implement its forward calculation

## Inputs

- moiety model JSON file (--models=<model_jsonfile>) defining moiety groups, moiety states, and isotopologue components for the single-tracer 13C model
- dataset JSON file (--datasets=<dataset_jsonfile>) with measured isotopologue profiles
- moiety state fractional abundances (optimized parameters) for each moiety group

## Outputs

- calculated relative isotopologue intensity profile Ix,calc per isotopologue mass shift x
- calculated_isotopologue_intensities.csv with calculated intensities for every isotopologue of the model

## How to apply

Load the moiety model definitions from the model JSON file (--models=<model_jsonfile>) and the measured isotopologue profiles from the dataset JSON file (--datasets=<dataset_jsonfile>), the two command-line inputs named in the moiety_modeling quickstart. For the single-tracer 13C model, enumerate all moiety state combinations across the moiety groups, and for each isotopologue mass shift x build IC_x, the set of components ic_v whose isotope content equals x. Compute each component ic_v as the product of the constituent moiety_state_{j,v_j} fractional abundances, then sum the IC_x components to obtain Ix,calc for that isotopologue. Normalize the calculated profile across isotopologues and write the results (e.g. calculated_isotopologue_intensities.csv). The rationale is that this same product-summation scheme was used with optimized moiety state values to generate both the single-tracer and multi-tracer 13C/18O simulated datasets, so a correct from-scratch implementation must reproduce the published package's forward calculation exactly.

## Related tools

- **moiety_modeling** (provides the moiety model representation and the reference forward calculation of Ix,calc; its model/dataset JSON inputs and quickstart CLI define the interface for this skill) — github.com/MoseleyBioinformaticsLab/moiety_modeling
- **numpy** (used to enumerate moiety state combinations and compute the products of moiety state fractional abundances and their sums for IC_x)
- **scipy** (Python library supplying additional optimization methods used with the moiety model in the framework) — github.com/scipy/scipy

## Examples

```
import numpy as np; IC_x = [np.prod([state_abund[(m, v)] for (m, v) in combo]) for combo in ic_components[x]]; I_x_calc = float(np.sum(IC_x))
```

## Evaluation signals

- Generate a reference profile with the published moiety_modeling package on identical model/dataset inputs and confirm the from-scratch calculated intensities match (output_matches_reference)
- Every IC_x component is a product of moiety_state_{j,v_j} fractional abundances, hence nonnegative and ≤ 1
- After normalization, the calculated relative intensities over all isotopologue mass shifts sum to 1
- The output CSV contains a correct field for every isotopologue of both the single-tracer 13C and multi-tracer 13C/18O models
- The same Eq. 1 code path, applied to optimized moiety state values, regenerates the single-tracer and multi-tracer simulated datasets

## Limitations

- Existing MAIMS software handles only 13C single isotope tracer data and does not address model selection; the moiety framework extends but does not replace curated-network flux methods
- When multiple moiety models are plausible, model selection (e.g. AICc ranking) is required before the forward calculation is interpretable in terms of relative metabolic flux
- Data quality requirements for model selection and validation of multiple isotope tracing model optimization remain future work

## Evidence

- [intro] we have developed a new moiety modeling framework for deconvoluting MS isotopologue profiles for both single and multiple-labeled SIRM MS datasets: "we have developed a new moiety modeling framework for deconvoluting MS isotopologue profiles for both single and multiple-labeled SIRM MS datasets"
- [full_text] The calculated relative intensity Ix,calc is the sum of IC_x components, each the product of moiety states: [evidence span withheld — anchor status: not_found]
- [methods] pickle_ for saving Python objects in a JSON serializable form and outputting to a file.: "pickle_ for saving Python objects in a JSON serializable form and outputting to a file."
- [readme] The moiety_modeling package provides facilities for moiety model representation, model optimization and model selection.: "`moiety_modeling` package provides facilities for moiety model representation, model optimization and model selection."
