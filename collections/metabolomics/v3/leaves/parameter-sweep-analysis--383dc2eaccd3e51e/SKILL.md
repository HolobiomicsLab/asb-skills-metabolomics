---
name: parameter-sweep-analysis
description: Use when a model's update rule contains an explicit mixing/weighting
  parameter (e.g. α∈[0,1] blending host-model bounds with aggregated microbiota fluxes)
  and you need the sensitivity of derived bounds or fluxes across that range.
when_to_use_negative:
- The goal is a single-point model modification with a fixed α rather than a sensitivity
  analysis across the α range
- The model's constraints contain no free weighting parameter to sweep (e.g. lower
  bounds are set directly by microbiota fluxes with no host/flux mixing)
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0605
tools:
- name: M2R
  role: Python add-on that applies the α-dependent update rule modifying exchange-reaction
    lower bounds using aggregated in- and out-fluxes from selected microbes
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads and manipulates the RECON3D human GEM and microbiota model during the
    α sweep
- name: Python
  role: Implementation language for driving the sweep and assembling the results table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:6cd8f1640a5e83a0ff88c16eb1f1b23751cd29c6462c8d9b890933033d3e3a6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# parameter-sweep-analysis

## Summary

Sweeps a model parameter (here the mixing weight α in M2R's exchange-reaction lower-bound update rule) over a defined range and records the resulting parameter-dependent model quantities, revealing how the parameter scales the trade-off between competing constraint sources.

## When to use

Use when a model's update rule contains an explicit mixing/weighting parameter (e.g. α∈[0,1] blending host-model bounds with aggregated microbiota fluxes) and you need the sensitivity of derived bounds or fluxes across that range.

## When NOT to use

- The goal is a single-point model modification with a fixed α rather than a sensitivity analysis across the α range
- The model's constraints contain no free weighting parameter to sweep (e.g. lower bounds are set directly by microbiota fluxes with no host/flux mixing)

## Inputs

- Human genome-scale metabolic model (GEM), e.g. RECON3D, loadable by cobrapy
- Paired gut microbiota metabolism model providing aggregated in- and out-fluxes (fi, fo) for selected microbes
- Grid of α values across [0, 1], including α=0.5 as the demonstrated midpoint

## Outputs

- Table of α–reaction–lower-bound records for every affected exchange reaction at every α value, saved as alpha_sweep_bounds.csv

## How to apply

Load the human GEM (RECON3D) and the paired gut microbiota metabolism model in Python with cobrapy. For each α on a grid across [0, 1] (including the demonstrated midpoint α=0.5), apply M2R's update rule LB_r := αLB_r + (1−α)fi − (1−α)fo, which modifies the lower bounds of exchange reactions using aggregated in- and out-fluxes from selected microbes. At every grid point, record the resulting lower bound of each affected exchange reaction. Assemble the α–reaction–lower-bound records into a table and save as alpha_sweep_bounds.csv. Rationale: sweeping α interpolates each bound between the fully host-scaled value (α=1) and the fully flux-scaled microbiota value (α=0), so the sweep quantifies how the weight scales the trade-off between the two constraint sources.

## Related tools

- **M2R** (Python add-on that applies the α-dependent update rule modifying exchange-reaction lower bounds using aggregated in- and out-fluxes from selected microbes) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads and manipulates the RECON3D human GEM and microbiota model during the α sweep)
- **Python** (Implementation language for driving the sweep and assembling the results table)

## Examples

```
python -c "import cobrapy; from m2r import update_bounds; [update_bounds(model, alpha=a, fi=fi, fo=fo) for a in [0,0.5,1]]; df.to_csv('alpha_sweep_bounds.csv')"
```

## Evaluation signals

- At α=1 every affected exchange-reaction lower bound equals the original host-model (host-scaled) value; at α=0 it equals the fully flux-scaled microbiota value fi − fo
- Each recorded bound equals αLB_r + (1−α)fi − (1−α)fo for its α, reaction, and aggregated fluxes (arithmetic re-derivation check)
- alpha_sweep_bounds.csv contains one record per (α, affected reaction) pair with no missing grid points or reactions
- Sweep includes the demonstrated midpoint α=0.5 and interpolates monotonically between the α=0 and α=1 endpoint bounds

## Limitations

- The sweep covers only exchange-reaction lower bounds; M2R's scope is modifying lower bounds of exchange reactions, not upper bounds or internal reaction constraints
- Results depend on the aggregated in- and out-fluxes from the selected microbes, so a different microbe selection changes the entire sweep
- No changelog found for M2R, so version-specific behavior of the update rule cannot be verified from the repository

## Evidence

- [full_text] M2R updates each affected reaction's lower bound as LB_r := αLB_r + (1−α)fi − (1−α)fo: [evidence span withheld — anchor status: not_found]
- [abstract] The idea behind the software is to modify the lower bounds: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] M2R is a Python add-on to cobrapy: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
