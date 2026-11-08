---
name: metabolite-exchange-flux-aggregation
description: 'Use when a human genome-scale metabolic model (e.g. RECON3D loaded in
  cobrapy as SBML) must be modified using gut microbiota metabolism: first aggregate
  and normalize the microbe''s external-reaction fluxes before setting exchange-reaction
  lower bounds.'
when_to_use_negative:
- The aim is to study a metabolic interaction without incorporating microbiota fluxes
  into the human GEM — this skill only supplies the aggregated/normalized fluxes M2R
  needs to modify exchange reaction lower bounds.
- 'The metabolites of interest are internal (non-boundary) reactions of the microbe:
  aggregation applies specifically to metabolites involved in the microbe''s external
  reactions.'
- Fluxes are already aggregated and normalized for the target exchange reactions —
  re-running aggregation would double-scale values relative to the maximum flux.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: M2R
  role: Python add-on to cobrapy that consumes the aggregated in- and out-fluxes to
    modify the lower bounds of exchange reactions in the human GEM (e.g. RECON3D)
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads the SBML microbiota model, runs FBA with the default LP solver, and
    provides the solution fluxes for external reactions that get aggregated
- name: Python
  role: Implementation language for the aggregation/normalization workflow and writing
    normalized_fluxes.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:a2460fbef0bb873f194d751d49b0dac48640074779fd175e4e7bf795ed073b3b
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
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolite-exchange-flux-aggregation

## Summary

Aggregate and normalize the in- and out-fluxes of boundary metabolites from an FBA solution of a gut microbiota metabolic model, producing per-metabolite flux values (normalized against the maximum flux, e.g. scaled to 1000) that M2R uses to modify lower bounds of exchange reactions in a human GEM such as RECON3D.

## When to use

Use when a human genome-scale metabolic model (e.g. RECON3D loaded in cobrapy as SBML) must be modified using gut microbiota metabolism: first aggregate and normalize the microbe's external-reaction fluxes before setting exchange-reaction lower bounds.

## When NOT to use

- The aim is to study a metabolic interaction without incorporating microbiota fluxes into the human GEM — this skill only supplies the aggregated/normalized fluxes M2R needs to modify exchange reaction lower bounds.
- The metabolites of interest are internal (non-boundary) reactions of the microbe: aggregation applies specifically to metabolites involved in the microbe's external reactions.
- Fluxes are already aggregated and normalized for the target exchange reactions — re-running aggregation would double-scale values relative to the maximum flux.

## Inputs

- Gut microbiota metabolic model for a selected microbe (SBML, loadable into cobrapy)
- Human genome-scale metabolic model (GEM) such as RECON3D to be modified

## Outputs

- Aggregated in- and out-flux value per boundary metabolite
- Normalized per-metabolite flux table written to normalized_fluxes.csv (divided by maximum aggregated flux, multiplied by e.g. 1000)
- Modified lower bounds of exchange reactions in the human GEM

## How to apply

Load the gut microbiota metabolic model (SBML) into cobrapy and identify the microbe's external (boundary) reactions and the metabolites participating in them. Run Flux Balance Analysis with cobrapy's default LP solver, then extract the in- and out-fluxes from the FBA solution for every metabolite involved in those external reactions. Aggregate the in- and out-fluxes per metabolite so each boundary metabolite carries a single combined flux value. Normalize each aggregated flux by dividing by the maximum aggregated flux and multiplying by a desired value, e.g. 1000. These normalized, aggregated in- and out-fluxes are the values M2R uses to modify the lower bounds of exchange reactions in the human GEM, controlling which metabolites enter and leave the reconstruction.

## Related tools

- **M2R** (Python add-on to cobrapy that consumes the aggregated in- and out-fluxes to modify the lower bounds of exchange reactions in the human GEM (e.g. RECON3D)) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads the SBML microbiota model, runs FBA with the default LP solver, and provides the solution fluxes for external reactions that get aggregated)
- **Python** (Implementation language for the aggregation/normalization workflow and writing normalized_fluxes.csv)

## Examples

```
import cobrapy; model = cobra.io.read_sbml_model('microbe.xml'); sol = model.optimize(); agg = sol.fluxes.groupby(boundary_metabolites).sum(); norm = agg / agg.max() * 1000; norm.to_csv('normalized_fluxes.csv')
```

## Evaluation signals

- Every metabolite appearing in the microbe's external reactions has exactly one aggregated flux value in the output table.
- The maximum normalized flux equals the desired value (e.g. 1000), since each aggregated flux is divided by the maximum aggregated flux before scaling.
- Aggregated in- and out-fluxes reproduce the per-metabolite values extracted from the cobrapy FBA solution for boundary reactions.
- The normalized fluxes correctly modify the lower bounds of exchange reactions in the human GEM when passed through M2R.
- normalized_fluxes.csv exists and contains one row per boundary metabolite with its normalized flux value.

## Limitations

- The aggregation is limited to metabolites involved in the microbe's external (boundary) reactions; internal microbial fluxes are not captured.
- Normalization is relative to the maximum aggregated flux, so the scale depends on the single largest flux in the FBA solution.
- No changelog found for the M2R repository, so version-specific behavior of the aggregation step is not documented.

## Evidence

- [methods] aggregate-then-normalize workflow grounded in FBA solution extraction: [evidence span withheld — anchor status: not_found]
- [abstract] purpose of modifying exchange reaction lower bounds: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] tool and model context for the aggregation step: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] effect on the metabolite pool entering and leaving the GEM: "M2R enables users to quickly and easily modify the pool of the metabolites that enter and leave the GEM, which is particularly important for those looking into an analysis of the metabolic"
