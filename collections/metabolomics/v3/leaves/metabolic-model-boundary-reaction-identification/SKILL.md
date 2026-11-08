---
name: metabolic-model-boundary-reaction-identification
description: Use when a gut microbiota metabolic model (SBML) has been loaded into
  cobrapy and you need per-metabolite boundary in/out fluxes from FBA to constrain
  a human GEM's exchange reactions with M2R.
when_to_use_negative:
- You only need internal reaction fluxes of the microbial model, not fluxes through
  its external/boundary reactions.
- The target is a microbial community model rather than modifying a human GEM's metabolite
  exchange pool via M2R.
- Fluxes are already aggregated and normalized (e.g. a previous M2R run's normalized_fluxes.csv
  exists) and only the lower-bound modification step remains.
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: M2R
  role: Consumes the normalized aggregated in-/out-fluxes to modify the lower bounds
    of exchange reactions in the human GEM (e.g. RECON3D)
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads the microbe SBML model, runs FBA with the default LP solver, and exposes
    exchange reactions and solution fluxes for aggregation
- name: Python
  role: Implementation language for the aggregation, normalization (divide by max
    flux, multiply by e.g. 1000) and CSV export steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:0308cc2d38695712206dbba8c33e5d4234d6b248f8d3b7791d616c77625c7440
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolic-model-boundary-reaction-identification

## Summary

Identifies a microbe's external (boundary) exchange reactions in an SBML genome-scale metabolic model loaded into cobrapy, extracts their metabolite in- and out-fluxes from an FBA solution, and aggregates and normalizes them per metabolite so they can modify exchange-reaction lower bounds in a human GEM (e.g. RECON3D) via M2R.

## When to use

Use when a gut microbiota metabolic model (SBML) has been loaded into cobrapy and you need per-metabolite boundary in/out fluxes from FBA to constrain a human GEM's exchange reactions with M2R.

## When NOT to use

- You only need internal reaction fluxes of the microbial model, not fluxes through its external/boundary reactions.
- The target is a microbial community model rather than modifying a human GEM's metabolite exchange pool via M2R.
- Fluxes are already aggregated and normalized (e.g. a previous M2R run's normalized_fluxes.csv exists) and only the lower-bound modification step remains.

## Inputs

- Gut microbiota metabolic model in SBML format (for the selected microbe)
- FBA solution from cobrapy's optimizer (flux distribution for the microbe model)
- Human genome-scale metabolic model such as RECON3D (target for M2R modification)

## Outputs

- Normalized per-metabolite in-/out-flux table written to normalized_fluxes.csv (aggregated fluxes scaled so the maximum equals the desired value, e.g. 1000)
- Modified human GEM with exchange-reaction lower bounds set from the normalized fluxes

## How to apply

Load the microbe's SBML model into cobrapy, then enumerate its external (boundary) reactions and the metabolites participating in them. Run Flux Balance Analysis with cobrapy's default LP optimizer and read the in- and out-fluxes from the solution for every boundary-reaction metabolite. Aggregate these in- and out-fluxes per metabolite so each boundary metabolite has a single combined flux value. Normalize by dividing each aggregated flux by the maximum aggregated flux and multiplying by the desired value (e.g. 1000), and write the normalized per-metabolite table to normalized_fluxes.csv. The rationale, per the M2R paper, is that these normalized fluxes are what M2R uses to modify the lower bounds of exchange reactions in a human GEM such as RECON3D, reshaping the pool of metabolites entering and leaving the model.

## Related tools

- **M2R** (Consumes the normalized aggregated in-/out-fluxes to modify the lower bounds of exchange reactions in the human GEM (e.g. RECON3D)) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads the microbe SBML model, runs FBA with the default LP solver, and exposes exchange reactions and solution fluxes for aggregation)
- **Python** (Implementation language for the aggregation, normalization (divide by max flux, multiply by e.g. 1000) and CSV export steps)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('microbe.xml'); sol = model.optimize(); # aggregate sol.fluxes over external-reaction metabolites, divide by max, multiply by 1000
```

## Evaluation signals

- Every metabolite taking part in the microbe's external reactions appears exactly once in the output table with a single combined in-/out-flux value.
- The maximum normalized aggregated flux equals the chosen target value (e.g. 1000); all other normalized fluxes are ≤ 1000.
- Normalized values match aggregated FBA fluxes divided by the maximum aggregated flux and multiplied by the chosen value.
- normalized_fluxes.csv is produced and its values, when passed to M2R, change the lower bounds of the corresponding exchange reactions in the human GEM.
- Fluxes in the table are traceable to the cobrapy FBA solution of the loaded SBML model (same objective value/solution).

## Limitations

- Only external (boundary) reaction metabolites are considered; internal microbial fluxes are ignored by design.
- Normalization is relative to the maximum aggregated flux, so an outlier boundary metabolite compresses all other normalized values.
- The method presupposes a successful FBA solution from cobrapy's default LP solver; infeasible or poorly scaled microbe models will propagate errors into the normalized fluxes.
- The article's extracted signals note no changelog is available for the repository, so version-specific behavior of M2R is not documented here.

## Evidence

- [abstract] M2R add-on and target models: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] Lower-bound modification from aggregated fluxes: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] Normalized fluxes modify the metabolite exchange pool: "M2R enables users to quickly and easily modify the pool of the metabolites that enter and leave the GEM"
