---
name: flux-normalization-scaling
description: Use when an FBA solution from a gut microbiota metabolic model must be
  converted into per-metabolite scaled fluxes to set exchange-reaction bounds in a
  human genome-scale metabolic model (e.g. Recon3D) via M2R.
when_to_use_negative:
- 'You need reaction-level or direction-resolved fluxes: aggregation collapses the
  in- and out-fluxes of each boundary metabolite into a single combined value.'
- Your goal is to modify constraints other than the lower bounds of exchange reactions
  — M2R's design only adjusts exchange-reaction lower bounds.
- The model is not a gut microbiota metabolic model loadable in cobrapy (e.g. non-SBML
  formats or non-metabolic reconstructions).
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3400
tools:
- name: M2R
  role: Python add-on that consumes the aggregated, normalized in- and out-fluxes
    to modify exchange-reaction lower bounds in the human GEM
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads the microbe's SBML model and runs FBA with the default LP solver to
    produce the raw in- and out-fluxes
- name: Python
  role: Implementation environment for the aggregation, normalization, and CSV export
    steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:28f3f01102f024ba475231166cacab1c7fced5dfcc31b797f3b864605666e7f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3400
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# flux-normalization-scaling

## Summary

A flux post-processing step from the M2R workflow: aggregate the FBA in- and out-fluxes of every metabolite participating in a gut microbe's external (boundary) reactions, then scale them so the maximum equals a desired value (e.g. 1000) for use in modifying exchange-reaction lower bounds in a human GEM such as Recon3D.

## When to use

Use when an FBA solution from a gut microbiota metabolic model must be converted into per-metabolite scaled fluxes to set exchange-reaction bounds in a human genome-scale metabolic model (e.g. Recon3D) via M2R.

## When NOT to use

- You need reaction-level or direction-resolved fluxes: aggregation collapses the in- and out-fluxes of each boundary metabolite into a single combined value.
- Your goal is to modify constraints other than the lower bounds of exchange reactions — M2R's design only adjusts exchange-reaction lower bounds.
- The model is not a gut microbiota metabolic model loadable in cobrapy (e.g. non-SBML formats or non-metabolic reconstructions).

## Inputs

- Gut microbiota metabolic model for a selected microbe (SBML, loadable via cobrapy)
- FBA solution fluxes for the microbe's external (boundary) reactions
- Human genome-scale metabolic model (e.g. Recon3D) whose exchange-reaction lower bounds will be modified

## Outputs

- normalized_fluxes.csv — one aggregated, normalized flux value per boundary metabolite (scaled so the maximum equals the desired value, e.g. 1000)
- Modified lower bounds of exchange reactions in the human GEM reflecting the aggregated in- and out-fluxes

## How to apply

Load the microbe's SBML model into cobrapy, identify its external (boundary) reactions and their metabolites, and run Flux Balance Analysis with the default LP solver. Extract the in- and out-fluxes from the FBA solution for every metabolite in those external reactions, then aggregate the in- and out-fluxes per metabolite so each boundary metabolite carries a single combined flux value. Normalize by dividing each aggregated value by the maximum aggregated flux and multiplying by the desired value, e.g. 1000. Write the normalized per-metabolite table to normalized_fluxes.csv; in M2R these aggregated, scaled fluxes are then used to modify the lower bounds of the exchange reactions in the human GEM.

## Related tools

- **M2R** (Python add-on that consumes the aggregated, normalized in- and out-fluxes to modify exchange-reaction lower bounds in the human GEM) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads the microbe's SBML model and runs FBA with the default LP solver to produce the raw in- and out-fluxes)
- **Python** (Implementation environment for the aggregation, normalization, and CSV export steps)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('gut_microbe.xml'); sol = model.optimize()
```

## Evaluation signals

- The maximum value in the normalized flux table equals the desired scaling value (e.g. exactly 1000 after division by the max aggregated flux and multiplication).
- Every metabolite participating in an external (boundary) reaction appears exactly once in normalized_fluxes.csv with a single aggregated value.
- Aggregated values reproduce the sign/magnitude relationships of the raw FBA in- and out-fluxes extracted from the cobrapy solution (normalization is a uniform scale factor of desired_value / max_flux).
- Re-running the workflow on the same SBML model and FBA solution yields an identical normalized_fluxes.csv (deterministic given the FBA optimum).
- Downstream check: the human GEM's exchange-reaction lower bounds reflect the scaled aggregated fluxes from the microbe model.

## Limitations

- Only the lower bounds of exchange reactions are modified; other model constraints are untouched by design.
- No changelog was found for the M2R repository, so version-specific behavior of the normalization step is not documented.
- The scaling factor depends on the maximum aggregated flux, so models with a single dominant boundary flux will compress the remaining metabolites' normalized values toward small magnitudes.

## Evidence

- [abstract] The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes.: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [methods] normalized by dividing aggregated in- and out-fluxes by the maximum flux value and multiplying by the desired value, e.g. 1000: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D.: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
