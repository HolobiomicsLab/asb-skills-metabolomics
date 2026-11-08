---
name: convex-combination-bound-recalculation
description: Use when a host genome-scale metabolic model (e.g. RECON3D in SBML) must
  be modified so metabolites produced or consumed by gut microbiota models alter the
  host exchange-reaction lower bounds, with a tunable weight α ∈ [0,1].
when_to_use_negative:
- The analysis concerns microbial–microbial interactions only, with no host GEM whose
  exchange lower bounds need modification
- A hard (non-weighted) replacement of exchange bounds is required rather than a convex
  combination controlled by α
- The target reactions contain no metabolites with identifiers present in the microbiota
  model, so no bounds would change
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
tools:
- name: M2R
  role: Python add-on that applies the convex-combination lower-bound update to host
    GEM exchange reactions using microbiota in/out fluxes
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Constraint-based modeling framework in which M2R runs, loading and saving
    the SBML GEMs
- name: Python
  role: Execution environment for the M2R/cobrapy workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:a4d6e86131d30b83cf7edd9e111d5c92bc29d6bd0e785c6cfcebc0486f024db8
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# convex-combination-bound-recalculation

## Summary

Recomputes the lower bounds of exchange reactions in a human GEM (e.g. RECON3D) as the convex combination LB_r := α·LB_r + (1−α)·f_i − (1−α)·f_o, blending the original bound with aggregated in- and out-fluxes from gut microbiota models, as implemented in the M2R cobrapy add-on.

## When to use

Use when a host genome-scale metabolic model (e.g. RECON3D in SBML) must be modified so metabolites produced or consumed by gut microbiota models alter the host exchange-reaction lower bounds, with a tunable weight α ∈ [0,1].

## When NOT to use

- The analysis concerns microbial–microbial interactions only, with no host GEM whose exchange lower bounds need modification
- A hard (non-weighted) replacement of exchange bounds is required rather than a convex combination controlled by α
- The target reactions contain no metabolites with identifiers present in the microbiota model, so no bounds would change

## Inputs

- Human genome-scale metabolic model (GEM) in SBML format, e.g. RECON3D
- Gut microbiota metabolic model(s) with exchange fluxes for shared metabolites
- User-specified weighting parameter α ∈ [0,1] (α = 0.5 in the demonstration)

## Outputs

- Modified GEM with updated exchange-reaction lower bounds, e.g. recon3d_modified.xml
- Per-reaction record of original lower bound LB_r and aggregated fluxes f_i, f_o used in the update

## How to apply

Load the human GEM (RECON3D) and the gut microbiota metabolic model with cobrapy/M2R in Python, then scan the host model for reactions containing at least one metabolite whose identifier is present in the microbiota model. For each matched metabolite, aggregate the in-flux f_i and out-flux f_o contributed by the selected microbiota reactions. For every collected host reaction r, apply the lower-bound update LB_r := α·LB_r + (1−α)·f_i − (1−α)·f_o, where α weights the original bound against microbiota fluxes (α = 0.5 in the paper's demonstration), modifying only reactions whose metabolites appear in the microbiota model. Verify that reactions not matching any microbiota metabolite retain their original lower bounds, then save the modified GEM (e.g. recon3d_modified.xml). Rationale: this convex combination lets the user continuously tune how strongly the microbially available metabolite pool replaces the default exchange constraints rather than hard-overwriting them.

## Related tools

- **M2R** (Python add-on that applies the convex-combination lower-bound update to host GEM exchange reactions using microbiota in/out fluxes) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Constraint-based modeling framework in which M2R runs, loading and saving the SBML GEMs)
- **Python** (Execution environment for the M2R/cobrapy workflow)

## Examples

```
import m2r; m2r.modify_bounds(model='recon3d.xml', microbiota='gut_model.xml', alpha=0.5, output='recon3d_modified.xml')
```

## Evaluation signals

- Recompute LB_r from stored original lower bounds and flux aggregates and confirm every updated bound equals α·LB_r + (1−α)·f_i − (1−α)·f_o exactly
- Confirm reactions whose metabolites do not match any microbiota-model identifier retain their original lower bounds unchanged
- Confirm only host reactions containing at least one metabolite present in the microbiota model were modified
- Check the modified model still loads in cobrapy and saves as valid SBML (recon3d_modified.xml)
- Sweep α across [0,1] and confirm the update interpolates monotonically between original bounds and pure microbiota-flux bounds

## Limitations

- The update only affects reactions containing metabolites shared with the microbiota model; metabolites absent from the microbiota model leave host bounds untouched
- The result depends on the user's choice of α (α = 0.5 in the demonstration) and on the selected set of microbiota reactions contributing f_i and f_o
- No changelog found for the M2R repository, limiting traceability of implementation changes

## Evidence

- [abstract] M2R lower-bound update rule: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] M2R purpose and scope: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [methods] Lower-bound update formula and validation from workflow card: [evidence span withheld — anchor status: not_found]
- [methods] Off-target invariance check: [evidence span withheld — anchor status: not_found]
