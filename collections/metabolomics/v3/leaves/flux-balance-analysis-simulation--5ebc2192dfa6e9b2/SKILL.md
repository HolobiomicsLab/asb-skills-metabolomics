---
name: flux-balance-analysis-simulation
description: Use when a gut microbiota SBML metabolic model must be simulated to derive
  boundary-metabolite fluxes that will modify exchange-reaction lower bounds of a
  human genome-scale metabolic model (e.g. RECON3D) via M2R/cobrapy.
when_to_use_negative:
- The model has no external/boundary (exchange) reactions annotated, so no per-metabolite
  in-/out-fluxes can be extracted.
- The goal is a standalone FBA of the human GEM itself (e.g. RECON3D) rather than
  deriving microbiota fluxes to modify its exchange-reaction lower bounds.
- The input is already a normalized flux table produced by a prior M2R run, so re-aggregation
  would double-transform the values.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: M2R
  role: Consumes the aggregated, normalized in-/out-fluxes from the FBA simulation
    to modify the lower bounds of exchange reactions in the human GEM
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads the SBML microbiota model, identifies external reactions, and runs FBA
    with the default LP solver to obtain the solution fluxes
- name: Python
  role: Environment in which M2R (a Python add-on to cobrapy) and the FBA/normalization
    workflow are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:f9e52f82fb8390a4e4320079dd2be249bbac9365545b7bab3cbb2159586b40fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# flux-balance-analysis-simulation

## Summary

Run Flux Balance Analysis on a gut microbiota metabolic model (SBML loaded into cobrapy) to obtain per-metabolite in- and out-fluxes at external reactions, which M2R aggregates and normalizes (divide by maximum flux, multiply by e.g. 1000) before writing them into the lower bounds of exchange reactions in a human GEM like RECON3D.

## When to use

Use when a gut microbiota SBML metabolic model must be simulated to derive boundary-metabolite fluxes that will modify exchange-reaction lower bounds of a human genome-scale metabolic model (e.g. RECON3D) via M2R/cobrapy.

## When NOT to use

- The model has no external/boundary (exchange) reactions annotated, so no per-metabolite in-/out-fluxes can be extracted.
- The goal is a standalone FBA of the human GEM itself (e.g. RECON3D) rather than deriving microbiota fluxes to modify its exchange-reaction lower bounds.
- The input is already a normalized flux table produced by a prior M2R run, so re-aggregation would double-transform the values.

## Inputs

- Gut microbiota metabolic model in SBML format (loadable by cobrapy)
- Desired normalization value for flux scaling (e.g. 1000)

## Outputs

- Aggregated in-/out-flux value per boundary (external-reaction) metabolite
- Normalized per-metabolite flux table (e.g. normalized_fluxes.csv)
- Modified exchange-reaction lower bounds in the human GEM (e.g. RECON3D)

## How to apply

Load the gut microbiota metabolic model (SBML) into cobrapy and identify its external (boundary) reactions together with the metabolites taking part in them. Run Flux Balance Analysis using cobrapy's optimizer with the default LP solver. From the FBA solution, extract the in- and out-fluxes for every metabolite participating in the external reactions, then aggregate the in- and out-fluxes per metabolite so each boundary metabolite has a single combined flux value. Normalize by dividing each aggregated value by the maximum aggregated flux and multiplying by the desired value (e.g. 1000), because M2R's design is to modify the lower bounds of exchange reactions using these aggregated fluxes. Write the normalized per-metabolite flux table (e.g. normalized_fluxes.csv) and use it to set exchange-reaction lower bounds in the human GEM.

## Related tools

- **M2R** (Consumes the aggregated, normalized in-/out-fluxes from the FBA simulation to modify the lower bounds of exchange reactions in the human GEM) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads the SBML microbiota model, identifies external reactions, and runs FBA with the default LP solver to obtain the solution fluxes)
- **Python** (Environment in which M2R (a Python add-on to cobrapy) and the FBA/normalization workflow are executed)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('microbe.xml'); solution = model.optimize(); fluxes = solution.fluxes
```

## Evaluation signals

- FBA returns an optimal solution from cobrapy's default LP solver (non-zero objective) before flux extraction.
- Every metabolite participating in an external/boundary reaction appears exactly once in the output table with a single aggregated flux value.
- After normalization, the maximum aggregated flux in the table equals the chosen desired value (e.g. 1000).
- The written normalized_fluxes.csv row set matches the set of metabolites found in the model's external reactions.
- Exchange-reaction lower bounds in the modified human GEM correspond to the sign and scaled magnitude of the microbiota in-/out-fluxes.

## Limitations

- Normalization is relative: all fluxes are scaled by the maximum aggregated flux and a user-chosen desired value (e.g. 1000), so absolute magnitudes from the FBA solution are not preserved.
- Aggregating in- and out-fluxes per metabolite into one combined value can obscure the direction of individual transport/exchange reactions.
- Correctness depends on the SBML model's external (boundary) reactions being properly identified; misclassification silently drops or misassigns fluxes.
- No changelog found for the M2R repository, so version-specific behavioral changes cannot be tracked from the provided context.

## Evidence

- [abstract] Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D.: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes.: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [other] fluxes of metabolites involved in the microbe's external reactions are read and aggregated, then normalized by dividing aggregated in- and out-fluxes by the maximum flux value: [evidence span withheld — anchor status: not_found]
