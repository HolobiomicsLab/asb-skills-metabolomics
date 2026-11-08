---
name: gut-microbiota-human-gem-integration
description: Use when a human GEM (e.g. RECON3D) loaded in cobrapy must be updated
  with in- and out-fluxes aggregated from one or more user-provided gut microbiota
  GEMs, to study the pool of metabolites exchanged between gut microbes and human
  cells.
when_to_use_negative:
- The analysis concerns a single-organism GEM with no gut microbiota models to incorporate
  — M2R specifically modifies exchange bounds using aggregated microbial in- and out-fluxes
- The goal is to modify internal reactions, upper bounds, or non-exchange reactions
  rather than the lower bounds of exchange reactions
- The model is not a human GEM loadable in cobrapy (e.g. a constraint-based model
  in a different format or framework incompatible with the M2R add-on)
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: M2R
  role: Python add-on to cobrapy that aggregates gut microbiota in- and out-fluxes
    and modifies the human GEM's exchange-reaction lower bounds
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads RECON3D and microbiota GEMs, and within which the M2R add-on modifies
    exchange-reaction lower bounds and exports the modified SBML
- name: Python
  role: Runtime environment in which M2R and cobrapy are executed end-to-end
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:53da386f4cabb1fd255c484a961ee056d5c6f62fe63f669cc8f51f6021ee3187
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gut-microbiota-human-gem-integration

## Summary

Integration of gut microbiota genome-scale metabolic models (GEMs) into a human GEM such as RECON3D using M2R, a Python add-on to cobrapy, which modifies the lower bounds of exchange reactions with aggregated microbial in- and out-fluxes weighted by a factor α (e.g. α = 0.5 in the paper's demonstration).

## When to use

Use when a human GEM (e.g. RECON3D) loaded in cobrapy must be updated with in- and out-fluxes aggregated from one or more user-provided gut microbiota GEMs, to study the pool of metabolites exchanged between gut microbes and human cells.

## When NOT to use

- The analysis concerns a single-organism GEM with no gut microbiota models to incorporate — M2R specifically modifies exchange bounds using aggregated microbial in- and out-fluxes
- The goal is to modify internal reactions, upper bounds, or non-exchange reactions rather than the lower bounds of exchange reactions
- The model is not a human GEM loadable in cobrapy (e.g. a constraint-based model in a different format or framework incompatible with the M2R add-on)

## Inputs

- RECON3D human genome-scale metabolic model (SBML loadable by cobrapy)
- One or more user-provided gut microbiota GEMs (SBML loadable by cobrapy)
- Selection of microbes whose fluxes will be incorporated
- Weighting factor alpha (e.g. 0.5) balancing original exchange-reaction lower bounds against aggregated microbial in-/out-fluxes

## Outputs

- Modified human GEM exported as recon3d_modified.xml (SBML loadable by cobrapy) with updated exchange-reaction lower bounds reflecting the aggregated in- and out-fluxes

## How to apply

Load the RECON3D human GEM and at least one user-provided gut microbiota GEM into cobrapy in Python, and select the microbes whose fluxes will be incorporated. Execute the M2R pipeline, which aggregates the in- and out-fluxes from the selected microbes and modifies the lower bounds of the human model's exchange reactions, balancing the original reaction lower bounds against the aggregated microbial fluxes via the weighting factor α (the paper's demonstration uses α = 0.5). Apply the aggregated fluxes through the M2R add-on to cobrapy, then export the modified human GEM as an SBML file (recon3d_modified.xml) loadable by cobrapy. Rationale: exchange-reaction lower bounds are the entry/exit points of the metabolite pool, so adjusting them encodes how gut microbial metabolism changes what enters and leaves the human GEM.

## Related tools

- **M2R** (Python add-on to cobrapy that aggregates gut microbiota in- and out-fluxes and modifies the human GEM's exchange-reaction lower bounds) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads RECON3D and microbiota GEMs, and within which the M2R add-on modifies exchange-reaction lower bounds and exports the modified SBML)
- **Python** (Runtime environment in which M2R and cobrapy are executed end-to-end)

## Examples

```
python -c "import cobra; m = cobra.io.read_sbml_model('recon3d_modified.xml'); print([r.lower_bound for r in m.exchanges[:5]])"
```

## Evaluation signals

- recon3d_modified.xml exists after the run
- recon3d_modified.xml loads in cobrapy without errors
- Exchange-reaction lower bounds in the modified model reflect the aggregated in- and out-fluxes from the selected gut microbiota models
- The α weighting (0.5 in the demonstration) is correctly applied as the balance between original lower bounds and aggregated microbial fluxes
- Biological plausibility of the modified metabolite pool is confirmed by expert review

## Limitations

- Any further biological plausibility of the modified metabolite pool is subject to expert review; the software guarantees only that the exchange lower bounds were updated
- Requires at least one user-provided gut microbiota GEM and a selection of microbes; results depend on which microbes are chosen and how their fluxes aggregate
- Designed for human GEMs such as RECON3D within the cobrapy ecosystem

## Evidence

- [abstract] M2R is a cobrapy add-on for gut-microbiota GEM integration: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] Mechanism: lower bounds of exchange reactions: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] Purpose: modifying the metabolite pool: "M2R enables users to quickly and easily modify the pool of the metabolites that enter and leave the GEM, which is particularly important for those looking into an analysis of the metabolic"
- [abstract] Availability and license: "M2R is freely available under an MIT License at https://github.com/e-weglarz-tomczak/m2r."
