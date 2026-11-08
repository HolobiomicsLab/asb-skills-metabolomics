---
name: exchange-reaction-lower-bound-update
description: Use when a host GEM (e.g., RECON3D in SBML) must be coupled to one or
  more gut microbiota metabolic models so that microbial metabolite fluxes shift the
  host model's exchange-reaction lower bounds.
when_to_use_negative:
- When the microbiota model shares no metabolite identifiers with the host GEM — no
  reactions would be matched and no bounds would change.
- When the goal is to modify upper bounds or internal (non-exchange) reactions — M2R's
  update applies only to the lower bounds of reactions containing microbiota metabolites.
- When all host reactions must remain untouched at their original bounds — at α =
  1 the update is fully host-scaled, so the microbiota fluxes would be effectively
  ignored.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: M2R
  role: Python add-on to cobrapy that executes the lower-bound update using aggregated
    in- and out-fluxes from selected microbes; MIT License
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads RECON3D and the microbiota GEMs, and exports the modified model as SBML
    (recon3d_modified.xml)
- name: Python
  role: Runtime environment for running the M2R five-step pipeline and the α-sweep
    over [0,1]
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:322bad938dcd405f1494ee8dbc46b4947d5713a3e342917c0eaa3db5d386fe29
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# exchange-reaction-lower-bound-update

## Summary

Method from the M2R paper (Bioinformatics 2021) for modifying the lower bounds of exchange reactions in a human genome-scale metabolic model (GEM) such as RECON3D, using aggregated in- and out-fluxes from gut microbiota models, via the M2R add-on to cobrapy.

## When to use

Use when a host GEM (e.g., RECON3D in SBML) must be coupled to one or more gut microbiota metabolic models so that microbial metabolite fluxes shift the host model's exchange-reaction lower bounds.

## When NOT to use

- When the microbiota model shares no metabolite identifiers with the host GEM — no reactions would be matched and no bounds would change.
- When the goal is to modify upper bounds or internal (non-exchange) reactions — M2R's update applies only to the lower bounds of reactions containing microbiota metabolites.
- When all host reactions must remain untouched at their original bounds — at α = 1 the update is fully host-scaled, so the microbiota fluxes would be effectively ignored.

## Inputs

- Human genome-scale metabolic model GEM (e.g., RECON3D) in SBML/XML loadable by cobrapy
- One or more user-provided gut microbiota GEMs loadable by cobrapy, with user-selected microbes whose fluxes will be incorporated
- User-specified weighting parameter α ∈ [0,1] (0.5 in the paper's demonstration)

## Outputs

- Modified human GEM as SBML, e.g., recon3d_modified.xml, with exchange-reaction lower bounds updated by the M2R rule
- For α-sensitivity analysis: a table of α–reaction–lower-bound records, e.g., alpha_sweep_bounds.csv

## How to apply

Load the human GEM (RECON3D) and the gut microbiota GEM(s) with cobrapy/M2R in Python, then scan the host model for reactions containing at least one metabolite whose identifier is present in the microbiota model. For each matched metabolite, aggregate the in-flux fi and out-flux fo contributed by the selected microbiota reactions across the five-step M2R pipeline. Apply the update LBr := α·LBr + (1−α)·fi − (1−α)·fo to every matched host reaction r, where α ∈ [0,1] weights the original bound against the microbiota fluxes (the paper's demonstration uses α = 0.5). Sweeping α from 0 to 1 interpolates each bound between the fully host-scaled value (α=1) and the fully flux-scaled value (α=0). Only reactions whose metabolites appear in the microbiota model are modified; all other reactions must retain their original lower bounds. Export the result as an SBML file (recon3d_modified.xml) loadable by cobrapy.

## Related tools

- **M2R** (Python add-on to cobrapy that executes the lower-bound update using aggregated in- and out-fluxes from selected microbes; MIT License) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads RECON3D and the microbiota GEMs, and exports the modified model as SBML (recon3d_modified.xml))
- **Python** (Runtime environment for running the M2R five-step pipeline and the α-sweep over [0,1])

## Evaluation signals

- Recompute each updated lower bound from stored original bounds and flux aggregates: every value must equal α·LBr + (1−α)·fi − (1−α)·fo exactly.
- Verify that reactions whose metabolites do not match any microbiota metabolite retain their original lower bounds (no off-target changes).
- Confirm recon3d_modified.xml exists and loads in cobrapy without errors.
- At α = 1 the sweep must reproduce the original host bounds; at α = 0 bounds must equal the fully microbiota-flux-scaled values (1−α)(fi − fo).
- Biological plausibility of the modified metabolite pool is subject to expert review, as noted in the source cards.

## Limitations

- The update only touches the lower bounds of reactions whose metabolites are present in the microbiota model; all other reactions are unchanged.
- Biological plausibility of the modified metabolite pool entering and leaving the GEM is flagged as requiring expert review.
- The repository README provided no changelog (missing signal: 'No changelog found').

## Evidence

- [abstract] M2R is a Python add-on to cobrapy for incorporating gut microbiota metabolism models into human GEMs such as RECON3D.: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] M2R modifies exchange-reaction lower bounds with aggregated microbial in- and out-fluxes.: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] M2R changes the pool of metabolites entering and leaving the GEM for host–microbiota interaction analysis.: "M2R enables users to quickly and easily modify the pool of the metabolites that enter and leave the GEM, which is particularly important for those looking into an analysis of the metabolic"
