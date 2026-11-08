---
name: gut-microbiota-human-metabolic-interaction-modeling
description: Use when a human GEM (e.g. RECON3D in cobrapy/SBML form) must be modified
  to reflect metabolite in- and out-fluxes from selected gut microbes, or when the
  α-weighted trade-off between host-scaled bounds and microbiota flux-scaled bounds
  must be swept and recorded.
when_to_use_negative:
- The model of interest is a microbiota model alone with no human GEM to modify —
  M2R targets human GEMs like RECON3D, not standalone microbial reconstructions
- The analysis requires changing reaction upper bounds or internal (non-exchange)
  reaction bounds — M2R works by modifying the lower bounds of exchange reactions
  only
- Microbial fluxes are not available or cannot be aggregated into fi/fo for the selected
  microbes, so the update rule cannot be evaluated
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3941
tools:
- name: M2R
  role: Python add-on to cobrapy that incorporates gut microbiota metabolism into
    the human GEM by modifying exchange-reaction lower bounds using aggregated in-
    and out-fluxes from selected microbes
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads RECON3D and the microbiota model in Python and provides the GEM objects
    whose exchange-reaction lower bounds M2R modifies
- name: Python
  role: Scripting environment in which the α sweep over [0,1] is executed and the
    α–reaction–lower-bound table is assembled
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:94bf5b544816d31a9460c0f121ee4cf46963f9f292812bdebe25444435ee9886
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
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gut-microbiota-human-metabolic-interaction-modeling

## Summary

A modeling skill for coupling gut microbiota metabolism models to human genome-scale metabolic models (GEMs) such as RECON3D using M2R, a Python add-on to cobrapy, which modifies exchange-reaction lower bounds from aggregated microbial in-/out-fluxes with a mixing weight α.

## When to use

Use when a human GEM (e.g. RECON3D in cobrapy/SBML form) must be modified to reflect metabolite in- and out-fluxes from selected gut microbes, or when the α-weighted trade-off between host-scaled bounds and microbiota flux-scaled bounds must be swept and recorded.

## When NOT to use

- The model of interest is a microbiota model alone with no human GEM to modify — M2R targets human GEMs like RECON3D, not standalone microbial reconstructions
- The analysis requires changing reaction upper bounds or internal (non-exchange) reaction bounds — M2R works by modifying the lower bounds of exchange reactions only
- Microbial fluxes are not available or cannot be aggregated into fi/fo for the selected microbes, so the update rule cannot be evaluated

## Inputs

- Human genome-scale metabolic model, e.g. RECON3D, loadable by cobrapy (SBML)
- Paired gut microbiota metabolism model with selected microbes
- Mixing weight α (scalar or grid over [0,1], e.g. including α=0.5)

## Outputs

- Modified cobrapy GEM whose exchange-reaction lower bounds are updated via LB_r := αLB_r + (1−α)fi − (1−α)fo
- Table of α–reaction–lower-bound records, e.g. alpha_sweep_bounds.csv

## How to apply

Load the fixed human GEM (RECON3D) and the paired gut microbiota metabolism model in Python with cobrapy. Compute the aggregated in-flux fi and out-flux fo for each affected exchange reaction from the selected microbes, then apply the M2R update rule LB_r := αLB_r + (1−α)fi − (1−α)fo so that α∈[0,1] interpolates each bound between the fully host-scaled value (α=1) and the fully flux-scaled microbiota value (α=0). Sweep α on a grid across [0,1], including the demonstrated midpoint α=0.5, and record the resulting lower bound of each affected exchange reaction at every α value. Assemble the α–reaction–lower-bound records into a table (e.g. alpha_sweep_bounds.csv) and inspect how exchange-reaction lower bounds vary across the range to characterize the host–microbiota metabolic interaction.

## Related tools

- **M2R** (Python add-on to cobrapy that incorporates gut microbiota metabolism into the human GEM by modifying exchange-reaction lower bounds using aggregated in- and out-fluxes from selected microbes) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads RECON3D and the microbiota model in Python and provides the GEM objects whose exchange-reaction lower bounds M2R modifies)
- **Python** (Scripting environment in which the α sweep over [0,1] is executed and the α–reaction–lower-bound table is assembled)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('RECON3D.xml'); from m2r import apply_microbiota; apply_microbiota(model, microbiota_model, alpha=0.5)
```

## Evaluation signals

- At α=1 every affected exchange-reaction lower bound equals the original host-scaled value LB_r, i.e. the microbiota term vanishes
- At α=0 every affected bound equals fi − fo, the fully flux-scaled microbiota value, confirming correct aggregation of in- and out-fluxes
- At the midpoint α=0.5 the bound equals 0.5·LB_r + 0.5·(fi − fo), providing a demonstrated interpolation check
- Only lower bounds of exchange reactions change; no other reaction bounds in the RECON3D GEM are altered
- The output table alpha_sweep_bounds.csv contains one record per (α, affected reaction) pair on the swept grid with finite lower-bound values

## Limitations

- Only lower bounds of exchange reactions are modified; the method does not touch upper bounds or internal reactions
- The result depends on the pool of selected microbes and their aggregated in-/out-fluxes fi and fo; changing the microbe selection changes all modified bounds
- The article's discussion contains no changelog ('No changelog found.'), so version-specific behavior of M2R cannot be verified from the provided context

## Evidence

- [abstract] M2R works by modifying the lower bounds of exchange reactions in the model using aggregated in- and out-fluxes from selected microbes.: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [other] M2R update rule interpolates bounds via mixing weight α.: [evidence span withheld — anchor status: not_found]
- [abstract] M2R is a Python add-on to cobrapy for human GEMs like RECON3D.: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] M2R modifies the metabolite pool entering and leaving the GEM.: "M2R enables users to quickly and easily modify the pool of the metabolites that enter and leave the GEM, which is particularly important for those looking into an analysis of the metabolic"
