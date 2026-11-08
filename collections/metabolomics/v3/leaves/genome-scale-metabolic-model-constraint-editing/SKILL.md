---
name: genome-scale-metabolic-model-constraint-editing
description: Use when a human GEM (e.g. RECON3D in SBML/XML) must be updated with
  flux information from gut microbiota models — i.e. host reactions containing metabolites
  present in the microbiota model need their lower bounds rewritten.
when_to_use_negative:
- The goal is to modify upper bounds, add/delete reactions, or edit host reactions
  whose metabolites are absent from the microbiota model — M2R only rewrites lower
  bounds of reactions sharing metabolites with the microbiota model
- The host GEM and microbiota model share no metabolite identifiers, so no reactions
  would be matched or updated
- The input is already a modified/feature-level model output rather than an original
  GEM plus microbiota flux data
edam_operation: http://edamontology.org/operation_0292
tools:
- name: M2R
  role: Python add-on to cobrapy that applies the lower-bound update rule LB_r :=
    α·LB_r + (1−α)·fi − (1−α)·fo to host GEM reactions using aggregated microbiota
    in-/out-fluxes
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads and manipulates the RECON3D GEM and microbiota model; M2R operates as
    an add-on on top of cobrapy
- name: Python
  role: Execution environment for running M2R/cobrapy scripts that scan, update, and
    save the GEM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:41835bdc260467ac7cd34de623368919854907bf9d366ad49884ccdf52b7261a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# genome-scale-metabolic-model-constraint-editing

## Summary

Edits the lower bounds of exchange reactions in a human genome-scale metabolic model (e.g. RECON3D) using aggregated in- and out-fluxes from gut microbiota metabolic models, via the M2R Python add-on to cobrapy. This couples host and microbiota metabolism so that microbially consumed/produced metabolites constrain what the human GEM can exchange.

## When to use

Use when a human GEM (e.g. RECON3D in SBML/XML) must be updated with flux information from gut microbiota models — i.e. host reactions containing metabolites present in the microbiota model need their lower bounds rewritten.

## When NOT to use

- The goal is to modify upper bounds, add/delete reactions, or edit host reactions whose metabolites are absent from the microbiota model — M2R only rewrites lower bounds of reactions sharing metabolites with the microbiota model
- The host GEM and microbiota model share no metabolite identifiers, so no reactions would be matched or updated
- The input is already a modified/feature-level model output rather than an original GEM plus microbiota flux data

## Inputs

- Human genome-scale metabolic model (e.g. RECON3D) as SBML/XML loadable by cobrapy
- Gut microbiota metabolic model (or set of microbe models) with reaction fluxes
- Weighting parameter α ∈ [0,1] (α = 0.5 used in the M2R demonstration)

## Outputs

- Modified human GEM with updated exchange-reaction lower bounds (saved as e.g. recon3d_modified.xml)
- Per-reaction record of original lower bound, aggregated in-flux fi, out-flux fo, and updated lower bound

## How to apply

Load the human GEM (e.g. RECON3D as XML with cobrapy) and the gut microbiota metabolic model in Python with the M2R add-on. Scan the host model and collect every reaction that contains at least one metabolite whose identifier is present in the microbiota model; leave all other reactions untouched. For each matched metabolite, aggregate the in-flux fi and out-flux fo contributed by the selected microbiota reactions, then apply the update LB_r := α·LB_r + (1−α)·fi − (1−α)·fo, where LB_r is the original lower bound of host reaction r and α ∈ [0,1] weights the original bound against the microbiota fluxes (α = 0.5 in the paper's demonstration). The rationale is to modify the pool of metabolites that enter and leave the GEM so host–microbiota metabolic interactions are captured in the exchange-reaction constraints. Save the modified GEM (e.g. recon3d_modified.xml) and validate by recomputing every updated bound from stored original bounds and flux aggregates.

## Related tools

- **M2R** (Python add-on to cobrapy that applies the lower-bound update rule LB_r := α·LB_r + (1−α)·fi − (1−α)·fo to host GEM reactions using aggregated microbiota in-/out-fluxes) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads and manipulates the RECON3D GEM and microbiota model; M2R operates as an add-on on top of cobrapy)
- **Python** (Execution environment for running M2R/cobrapy scripts that scan, update, and save the GEM)

## Evaluation signals

- Recompute each updated lower bound from the stored original LB_r and flux aggregates and confirm it equals α·LB_r + (1−α)·fi − (1−α)·fo exactly (with α = 0.5 in the demonstration setting)
- Diff the modified GEM against the original RECON3D and confirm reactions not matching any microbiota metabolite retain their original lower bounds (no off-target changes)
- Confirm only reactions containing at least one metabolite present in the microbiota model were modified
- Verify the saved recon3d_modified.xml loads in cobrapy without schema errors and preserves all non-bound model attributes
- Check that all updated lower bounds remain within valid flux-bounds semantics (LB ≤ UB) after the update

## Limitations

- Only lower bounds of exchange reactions are modified; upper bounds and other model components are not touched
- The update depends on the metabolite identifiers shared between host and microbiota models — unmatched metabolites contribute no constraint
- The result depends on the user-specified weighting α; the choice of α (e.g. 0.5 in the demonstration) directly scales how strongly microbiota fluxes override the original bounds
- No changelog was found for the tool, so version-specific behavior changes are not documented

## Evidence

- [abstract] M2R is a Python add-on to cobrapy for incorporating gut microbiota metabolism models into human GEMs such as RECON3D.: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] M2R modifies lower bounds of exchange reactions using aggregated microbiota in- and out-fluxes.: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] M2R changes the metabolite pool entering and leaving the GEM, relevant for host–microbiota metabolic interaction studies.: "M2R enables users to quickly and easily modify the pool of the metabolites that enter and leave the GEM, which is particularly important for those looking into an analysis of the metabolic"
- [abstract] M2R availability and license.: "M2R is freely available under an MIT License at https://github.com/e-weglarz-tomczak/m2r."
