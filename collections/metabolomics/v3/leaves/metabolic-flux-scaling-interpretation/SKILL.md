---
name: metabolic-flux-scaling-interpretation
description: Use when you have a human GEM paired with microbiota-derived aggregated
  in-/out-fluxes (fi, fo) and need to characterize how the α mixing weight scales
  the resulting exchange-reaction lower bounds across [0, 1].
when_to_use_negative:
- No microbiota-derived aggregated fluxes (fi, fo) are available for the selected
  microbes, so the update rule cannot be evaluated at any α
- The goal is to modify model upper bounds or reaction stoichiometry rather than exchange-reaction
  lower bounds — M2R's rule only updates lower bounds
- The target model is not a human GEM compatible with cobrapy (e.g. constraint-based
  microbiota-only model), where the host-bound term αLB_r has no meaning
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2250
tools:
- name: M2R
  role: Provides the lower-bound update rule LB_r := αLB_r + (1−α)fi − (1−α)fo that
    modifies exchange-reaction lower bounds in the human GEM using aggregated microbiota
    in-/out-fluxes; applied at each α on the sweep grid.
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads and manipulates the human GEM (RECON3D), exposing reaction lower_bound
    attributes that the M2R rule rewrites at each α value.
- name: Python
  role: Implements the α grid sweep, computes updated lower bounds per reaction, and
    assembles the α–reaction–bound records into alpha_sweep_bounds.csv.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:419dad9812e3c93820deabedf74ddbf84932a6923b985284a13f8c8b8c4ff4fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolic-flux-scaling-interpretation

## Summary

Sweeps the mixing weight α in M2R's lower-bound update rule (LB_r := αLB_r + (1−α)fi − (1−α)fo) to quantify how exchange-reaction lower bounds in a human GEM such as RECON3D interpolate between the host-model bound (α=1) and the aggregated microbiota in-/out-flux value (α=0).

## When to use

Use when you have a human GEM paired with microbiota-derived aggregated in-/out-fluxes (fi, fo) and need to characterize how the α mixing weight scales the resulting exchange-reaction lower bounds across [0, 1].

## When NOT to use

- No microbiota-derived aggregated fluxes (fi, fo) are available for the selected microbes, so the update rule cannot be evaluated at any α
- The goal is to modify model upper bounds or reaction stoichiometry rather than exchange-reaction lower bounds — M2R's rule only updates lower bounds
- The target model is not a human GEM compatible with cobrapy (e.g. constraint-based microbiota-only model), where the host-bound term αLB_r has no meaning

## Inputs

- Human genome-scale metabolic model (GEM), e.g. RECON3D, loadable by cobrapy
- Paired gut microbiota metabolism model yielding aggregated in-fluxes (fi) and out-fluxes (fo) for selected microbes
- List/grid of α values in [0, 1] (including α = 0.5)
- Identifiers of the exchange reactions affected by the M2R update

## Outputs

- alpha_sweep_bounds.csv — one record per (α, reaction) with the updated lower bound LB_r
- Per-reaction sensitivity profile of lower bounds across the α grid

## How to apply

Load the fixed human GEM (e.g. RECON3D) and the paired gut microbiota metabolism model in Python with cobrapy, then identify the affected exchange reactions whose lower bounds M2R modifies using aggregated in- and out-fluxes (fi, fo) from selected microbes. Define a grid of α values across [0, 1], making sure to include α = 0.5 (the demonstrated midpoint) and the two endpoints α = 0 and α = 1. At each α, apply the M2R update rule LB_r := αLB_r + (1−α)fi − (1−α)fo to every affected exchange reaction and record the resulting lower bound. The rationale is that α linearly interpolates each bound between the fully host-scaled original value (α=1) and the fully flux-scaled microbiota value (α=0), so the sweep exposes how strongly each exchange reaction is driven by microbiota flux versus the host reconstruction. Assemble the α–reaction–lower-bound records into a table (alpha_sweep_bounds.csv) and inspect per-reaction sensitivity, e.g. bounds that change rapidly near α = 0 are dominated by microbiota fluxes.

## Related tools

- **M2R** (Provides the lower-bound update rule LB_r := αLB_r + (1−α)fi − (1−α)fo that modifies exchange-reaction lower bounds in the human GEM using aggregated microbiota in-/out-fluxes; applied at each α on the sweep grid.) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads and manipulates the human GEM (RECON3D), exposing reaction lower_bound attributes that the M2R rule rewrites at each α value.)
- **Python** (Implements the α grid sweep, computes updated lower bounds per reaction, and assembles the α–reaction–bound records into alpha_sweep_bounds.csv.)

## Examples

```
bounds = {a: {r.id: a*model.reactions.get_by_id(r.id).lower_bound + (1-a)*(fi[r.id] - fo[r.id]) for r in affected} for a in alphas}
```

## Evaluation signals

- At α = 1, every affected exchange reaction's lower bound equals the original host-model lower bound LB_r exactly
- At α = 0, every affected bound equals the aggregated microbiota value fi − fo exactly
- For each reaction, the bound varies linearly in α across the sweep grid — no jumps or non-monotone behavior
- The α = 0.5 record equals the arithmetic midpoint of the α = 0 and α = 1 bounds for each reaction
- alpha_sweep_bounds.csv contains one row per (α, reaction) pair with no missing bounds, and bound values stay within the interval spanned by the two endpoint values

## Limitations

- The update only affects lower bounds of exchange reactions; other model constraints are untouched, so the sweep does not characterize changes to capacities on the uptake side of the bound alone
- The interpolation is only as good as the aggregated fi/fo fluxes from the selected microbes — errors in microbiota flux aggregation propagate linearly through all α values
- Demonstrated specifically on a human GEM (RECON3D) paired with gut microbiota models; transferability to other host models is not established
- No changelog found for the M2R repository, so version-specific behavior of the update rule is unverified

## Evidence

- [abstract] The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes.: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [other] LB_r := αLB_r + (1−α)fi − (1−α)fo interpolates between host-scaled (α=1) and flux-scaled (α=0) values.: [evidence span withheld — anchor status: not_found]
- [abstract] M2R is a Python add-on to cobrapy incorporating gut microbiota metabolism models into human GEMs like RECON3D.: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
