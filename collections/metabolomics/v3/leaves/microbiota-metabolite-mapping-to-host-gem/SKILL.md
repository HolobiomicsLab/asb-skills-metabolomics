---
name: microbiota-metabolite-mapping-to-host-gem
description: Use when a human GEM (e.g., RECON3D) must be modified with flux information
  from gut microbiota models, updating exchange-reaction lower bounds of reactions
  that contain metabolites present in the microbiota model.
when_to_use_negative:
- The microbiota model shares no metabolite identifiers with the host GEM, so no reactions
  can be matched and no bounds can be updated.
- The goal is to add reactions or modify upper bounds of the host GEM rather than
  adjust the lower bounds of exchange reactions.
- The task is simulation of the gut microbiota model alone, without integration into
  a human GEM like RECON3D.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: M2R
  role: Python add-on that performs the lower-bound update of exchange reactions using
    aggregated microbiota in- and out-fluxes
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads and manipulates the human GEM (RECON3D) and microbiota model; M2R is
    an add-on to it
- name: Python
  role: Execution environment in which cobrapy and M2R are run to load, scan, update,
    and save the GEM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:89da647e331b507d3ba7c847dcee0e5d1a716830db6fa4b99279fe2eefe18091
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# microbiota-metabolite-mapping-to-host-gem

## Summary

Applies M2R (a Python add-on to cobrapy) to incorporate gut microbiota metabolic models into a human genome-scale metabolic model (GEM) such as RECON3D by rewriting exchange-reaction lower bounds with aggregated microbiota in/out-fluxes.

## When to use

Use when a human GEM (e.g., RECON3D) must be modified with flux information from gut microbiota models, updating exchange-reaction lower bounds of reactions that contain metabolites present in the microbiota model.

## When NOT to use

- The microbiota model shares no metabolite identifiers with the host GEM, so no reactions can be matched and no bounds can be updated.
- The goal is to add reactions or modify upper bounds of the host GEM rather than adjust the lower bounds of exchange reactions.
- The task is simulation of the gut microbiota model alone, without integration into a human GEM like RECON3D.

## Inputs

- Human genome-scale metabolic model (GEM), e.g., RECON3D in SBML/XML format
- Gut microbiota metabolic model(s) with metabolite identifiers mappable to the host GEM
- User-specified weighting parameter α ∈ [0,1] (α = 0.5 in the demonstration)

## Outputs

- Modified human GEM saved as recon3d_modified.xml with updated exchange-reaction lower bounds
- Per-reaction record of original lower bounds LBr and aggregated microbiota in-flux fi / out-flux fo used in the update

## How to apply

Load the human GEM (e.g., RECON3D) and the gut microbiota metabolic model with cobrapy/M2R in Python. Scan the host model and collect every reaction that contains at least one metabolite whose identifier is present in the microbiota model. For each matched metabolite, aggregate the in-flux fi and out-flux fo contributed by the selected microbiota reactions. Then apply M2R's lower-bound update LBr := α·LBr + (1−α)·fi − (1−α)·fo, where LBr is the original lower bound and α ∈ [0,1] (α = 0.5 in the paper's demonstration) weights the original bound against the microbiota fluxes. Verify that reactions not matching any microbiota metabolite retain their original lower bounds, then save the modified model as recon3d_modified.xml. The rationale is that only the pool of metabolites entering and leaving the GEM changes, capturing metabolic interaction between gut microbiota and human cells without altering the host reaction network itself.

## Related tools

- **M2R** (Python add-on that performs the lower-bound update of exchange reactions using aggregated microbiota in- and out-fluxes) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads and manipulates the human GEM (RECON3D) and microbiota model; M2R is an add-on to it)
- **Python** (Execution environment in which cobrapy and M2R are run to load, scan, update, and save the GEM)

## Evaluation signals

- Recompute every updated bound from stored original lower bounds and flux aggregates and confirm it equals α·LBr + (1−α)·fi − (1−α)·fo exactly.
- Confirm that reactions not matching any microbiota metabolite retain their original lower bounds (no off-target changes).
- Confirm that only reactions containing at least one metabolite identifier present in the microbiota model were modified.
- Verify the saved recon3d_modified.xml loads cleanly in cobrapy and yields a feasible optimization after the bound updates.

## Limitations

- M2R modifies only the lower bounds of exchange reactions; it does not alter upper bounds or add new reactions.
- The update depends on metabolite identifier matching between the microbiota model and the host GEM; unmatched metabolites contribute nothing.
- The weighting parameter α is user-specified (0.5 in the demonstration) and its choice directly scales how strongly microbiota fluxes override original bounds.
- No changelog found for the M2R repository, so version-specific behavior may be undocumented.

## Evidence

- [abstract] M2R as a Python add-on to cobrapy for GEM integration: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] Lower-bound modification mechanism: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] Purpose: modifying the metabolite pool entering and leaving the GEM: "M2R enables users to quickly and easily modify the pool of the metabolites that enter and leave the GEM, which is particularly important for those looking into an analysis of the metabolic"
