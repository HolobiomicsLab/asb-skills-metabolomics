---
name: microbiota-metabolic-model-parsing
description: Use when you have a gut microbiota metabolic model (SBML) and need its
  per-metabolite boundary fluxes aggregated and normalized (scaled to e.g. 1000) to
  modify exchange-reaction lower bounds in a human GEM like RECON3D.
when_to_use_negative:
- You need to run FBA on the human GEM itself (e.g. RECON3D) rather than extract boundary
  fluxes from a microbiota model — this skill only processes the microbe's model.
- Your microbiota model has no external (boundary) reactions to aggregate — the aggregation/normalization
  step is undefined in that case.
- You want to modify the human GEM directly without microbe-derived fluxes — plain
  cobrapy editing, not this skill, is the right approach.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: M2R
  role: Python add-on that consumes the aggregated in- and out-fluxes to modify the
    lower bounds of exchange reactions in the human GEM
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads the SBML microbiota model and runs Flux Balance Analysis with the default
    LP solver to obtain reaction fluxes
- name: Python
  role: Implementation language for the parsing, aggregation, and normalization workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:0ef5a9c2859bc7341af17039d4cd0f8623b69bd785266c35dc4d1503b98c14fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# microbiota-metabolic-model-parsing

## Summary

Parse a gut microbiota genome-scale metabolic model (SBML) with cobrapy, run FBA, and aggregate/normalize the in- and out-fluxes of every metabolite participating in the microbe's external (boundary) reactions so they can be used to modify a human GEM such as RECON3D via M2R.

## When to use

Use when you have a gut microbiota metabolic model (SBML) and need its per-metabolite boundary fluxes aggregated and normalized (scaled to e.g. 1000) to modify exchange-reaction lower bounds in a human GEM like RECON3D.

## When NOT to use

- You need to run FBA on the human GEM itself (e.g. RECON3D) rather than extract boundary fluxes from a microbiota model — this skill only processes the microbe's model.
- Your microbiota model has no external (boundary) reactions to aggregate — the aggregation/normalization step is undefined in that case.
- You want to modify the human GEM directly without microbe-derived fluxes — plain cobrapy editing, not this skill, is the right approach.

## Inputs

- Gut microbiota metabolic model in SBML format for the selected microbe (loadable by cobrapy)
- Human genome-scale metabolic model (GEM) such as RECON3D to be modified downstream by M2R

## Outputs

- normalized_fluxes.csv: per-metabolite table of aggregated in- and out-fluxes, normalized by dividing by the maximum aggregated flux and multiplying by 1000
- Normalized flux values suitable for modifying the lower bounds of exchange reactions in the human GEM via M2R

## How to apply

Load the microbe's SBML model into cobrapy and identify the external (boundary) reactions together with the metabolites they involve. Run Flux Balance Analysis with cobrapy's default LP optimizer and extract the in- and out-fluxes from the solution for every metabolite taking part in those external reactions. Aggregate the in- and out-fluxes per metabolite so each boundary metabolite carries a single combined flux value. Normalize by dividing each aggregated flux by the maximum aggregated flux and multiplying by the desired value (e.g. 1000). These normalized fluxes are what M2R uses to modify the lower bounds of exchange reactions in the human GEM, adjusting the pool of metabolites that enter and leave it.

## Related tools

- **M2R** (Python add-on that consumes the aggregated in- and out-fluxes to modify the lower bounds of exchange reactions in the human GEM) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads the SBML microbiota model and runs Flux Balance Analysis with the default LP solver to obtain reaction fluxes)
- **Python** (Implementation language for the parsing, aggregation, and normalization workflow)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('microbe.xml'); sol = model.optimize(); # aggregate exchange in-/out-fluxes per metabolite, divide by max, multiply by 1000 -> normalized_fluxes.csv
```

## Evaluation signals

- Every metabolite participating in an external (boundary) reaction of the microbe's model appears as a row in normalized_fluxes.csv.
- Each boundary metabolite has a single aggregated flux value, not separate raw in- and out-flux entries.
- The maximum absolute normalized flux value in the table equals the scaling target (e.g. 1000), since all values are divided by the maximum aggregated flux.
- Normalized fluxes are signed (in-fluxes vs out-fluxes preserved) so they can be mapped onto exchange-reaction lower bounds by M2R.
- The downstream modified human GEM (e.g. RECON3D) has exchange-reaction lower bounds consistent with the normalized microbe fluxes.

## Limitations

- Normalization rescales all fluxes relative to the single maximum aggregated flux, so flux ratios below the maximum are compressed proportionally to the chosen target value (e.g. 1000).
- Only metabolites involved in the microbe's external reactions are considered; intracellular microbial fluxes are not exported.
- No changelog found for the M2R repository, so version-specific behavior may be untracked.

## Evidence

- [abstract] M2R as Python cobrapy add-on for human GEMs: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] M2R modifies exchange-reaction lower bounds using aggregated fluxes: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [other] Aggregation per boundary metabolite: [evidence span withheld — anchor status: not_found]
- [other] Normalization to target value 1000: [evidence span withheld — anchor status: not_found]
