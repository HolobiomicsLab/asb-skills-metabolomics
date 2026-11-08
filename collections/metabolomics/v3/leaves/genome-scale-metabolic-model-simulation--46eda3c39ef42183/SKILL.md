---
name: genome-scale-metabolic-model-simulation
description: Use when a human GEM (RECON3D or similar, SBML/cobrapy-loadable) must
  be simulated with the metabolite pool adjusted to gut microbiota fluxes, or when
  the α-weighted trade-off between host bounds and microbial in-/out-fluxes must be
  swept.
when_to_use_negative:
- The input is not a genome-scale metabolic reconstruction (e.g. you have only a microbiota
  model and no paired human GEM like RECON3D to modify)
- You need to modify upper bounds, internal (non-exchange) reactions, or perform other
  constraint edits outside the exchange-reaction lower-bound update rule that M2R
  implements
- Biological plausibility of the modified metabolite pool is the question itself —
  the article leaves that to expert review; M2R only guarantees the arithmetic of
  the bound update
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_2258
tools:
- name: M2R
  role: Python add-on to cobrapy that executes the five-step pipeline aggregating
    microbial in-/out-fluxes and updating exchange-reaction lower bounds with weight
    α
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads RECON3D and the gut microbiota GEMs, and validates that the exported
    recon3d_modified.xml reloads without errors with updated exchange bounds
- name: Python
  role: Language environment in which the M2R add-on and cobrapy are run, including
    the α grid sweep and CSV assembly
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:1a68e4d4e77428a5de5391d869b27ad481cb955aa73258006d197c5e7e055bc6
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# genome-scale-metabolic-model-simulation

## Summary

Simulate and modify a human genome-scale metabolic model (e.g. RECON3D) in cobrapy/Python using the M2R add-on, which incorporates gut microbiota metabolism by updating exchange-reaction lower bounds with aggregated microbial in-/out-fluxes.

## When to use

Use when a human GEM (RECON3D or similar, SBML/cobrapy-loadable) must be simulated with the metabolite pool adjusted to gut microbiota fluxes, or when the α-weighted trade-off between host bounds and microbial in-/out-fluxes must be swept.

## When NOT to use

- The input is not a genome-scale metabolic reconstruction (e.g. you have only a microbiota model and no paired human GEM like RECON3D to modify)
- You need to modify upper bounds, internal (non-exchange) reactions, or perform other constraint edits outside the exchange-reaction lower-bound update rule that M2R implements
- Biological plausibility of the modified metabolite pool is the question itself — the article leaves that to expert review; M2R only guarantees the arithmetic of the bound update

## Inputs

- Human genome-scale metabolic model (RECON3D) in SBML format loadable by cobrapy
- One or more user-provided gut microbiota GEMs in SBML format loadable by cobrapy
- Mixing weight α (scalar in [0,1], or a grid across [0,1] including α=0.5) and the list of selected microbes whose fluxes are aggregated

## Outputs

- Modified human GEM exported as recon3d_modified.xml (SBML loadable by cobrapy) with updated exchange-reaction lower bounds
- Table of α–reaction–lower-bound records (alpha_sweep_bounds.csv) from the α sensitivity sweep

## How to apply

Load the fixed human GEM (RECON3D) and the paired gut microbiota metabolism GEMs into cobrapy in Python, then run the M2R five-step pipeline, selecting the microbes whose fluxes will be incorporated. M2R aggregates the in- and out-fluxes (fi, fo) from the selected microbes and modifies the lower bounds of the model's exchange reactions according to LB_r := αLB_r + (1−α)fi − (1−α)fo; the paper's demonstration uses α = 0.5 as the midpoint balancing host bounds against microbial fluxes, while sweeping α over [0, 1] interpolates each bound between the fully host-scaled value (α=1) and the fully flux-scaled microbiota value (α=0). Record each affected exchange reaction's lower bound across the chosen α grid and export the modified model as SBML (recon3d_modified.xml) loadable by cobrapy. Correctness is judged by the exported model reloading without errors in cobrapy and its exchange-reaction lower bounds reflecting the aggregated in-/out-fluxes.

## Related tools

- **M2R** (Python add-on to cobrapy that executes the five-step pipeline aggregating microbial in-/out-fluxes and updating exchange-reaction lower bounds with weight α) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads RECON3D and the gut microbiota GEMs, and validates that the exported recon3d_modified.xml reloads without errors with updated exchange bounds)
- **Python** (Language environment in which the M2R add-on and cobrapy are run, including the α grid sweep and CSV assembly)

## Examples

```
from cobra.io import read_sbml_model; from m2r import M2R; M2R(model=read_sbml_model('RECON3D.xml'), microbiota=[read_sbml_model('gut_gem.xml')], alpha=0.5).run(); write_sbml_model(modified, 'recon3d_modified.xml')
```

## Evaluation signals

- recon3d_modified.xml exists and loads in cobrapy without errors
- Exchange-reaction lower bounds in the exported model equal αLB_r + (1−α)fi − (1−α)fo for the selected microbes' aggregated in-/out-fluxes
- α=1 reproduces the original host-model lower bounds exactly; α=0 yields the fully flux-scaled microbiota bounds (interpolation endpoints)
- alpha_sweep_bounds.csv contains one record per (α, affected exchange reaction) pair with no missing reactions on the grid
- The aggregated in-/out-fluxes match the fluxes of the microbes selected by the user, not the full microbiota model

## Limitations

- M2R only modifies the lower bounds of exchange reactions; other constraint types are out of scope
- Biological plausibility of the modified metabolite pool after the update is subject to expert review and is not guaranteed by the tool
- The result depends on the user's selection of microbes and on the α weight; a single α (e.g. the demonstrated 0.5) is an arbitrary midpoint unless justified by a sensitivity sweep
- No changelog was found, so version-specific behavior of the repository is not documented

## Evidence

- [other] M2R updates each affected reaction's lower bound as LB_r := αLB_r + (1−α)fi − (1−α)fo, so sweeping α from 0 to 1 interpolates each bound between the fully host-scaled value (α=1) and the fully flux-scaled microbiota value (α=0).: [evidence span withheld — anchor status: not_found]
- [abstract] The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes.: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D.: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [other] verify that recon3d_modified.xml exists, loads in cobrapy without errors, and that its exchange-reaction lower bounds reflect the aggregated in- and out-fluxes from the selected gut microbiota models: [evidence span withheld — anchor status: not_found]
