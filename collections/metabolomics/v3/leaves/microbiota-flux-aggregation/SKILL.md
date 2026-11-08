---
name: microbiota-flux-aggregation
description: Use when a human GEM (e.g. RECON3D in SBML/cobrapy format) must be conditioned
  on gut microbiota metabolism — i.e. when studying metabolic interaction between
  gut microbes and human cells and exchange-reaction lower bounds need updating from
  microbial fluxes.
when_to_use_negative:
- The model of interest is not a human GEM whose exchange reactions should be conditioned
  on gut microbial metabolism — M2R targets human GEMs like RECON3D.
- You need de novo simulation of microbiota community flux rather than modifying an
  existing human GEM's exchange bounds from selected microbes.
- No gut microbiota metabolic models are available as input — the aggregation step
  has nothing to incorporate.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3895
tools:
- name: M2R
  role: Python add-on to cobrapy that aggregates in- and out-fluxes from selected
    gut microbiota models and modifies the lower bounds of exchange reactions in the
    human GEM
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: cobrapy
  role: Loads RECON3D and the microbiota GEMs, hosts the M2R add-on, and re-loads
    the exported SBML for validation
- name: Python
  role: Runtime environment in which the M2R five-step pipeline is executed end-to-end
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:89ba2080c839b80925aebcb7436e0a24691b36dddc0f3bfd658e847864c399d5
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3895
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# microbiota-flux-aggregation

## Summary

Aggregate in- and out-fluxes from user-selected gut microbiota GEMs and apply them to modify the lower bounds of exchange reactions in a human genome-scale metabolic model (e.g. RECON3D), using the M2R add-on to cobrapy. This changes the pool of metabolites entering and leaving the human GEM to reflect gut microbial metabolism.

## When to use

Use when a human GEM (e.g. RECON3D in SBML/cobrapy format) must be conditioned on gut microbiota metabolism — i.e. when studying metabolic interaction between gut microbes and human cells and exchange-reaction lower bounds need updating from microbial fluxes.

## When NOT to use

- The model of interest is not a human GEM whose exchange reactions should be conditioned on gut microbial metabolism — M2R targets human GEMs like RECON3D.
- You need de novo simulation of microbiota community flux rather than modifying an existing human GEM's exchange bounds from selected microbes.
- No gut microbiota metabolic models are available as input — the aggregation step has nothing to incorporate.

## Inputs

- Human genome-scale metabolic model (GEM), e.g. RECON3D, loadable by cobrapy
- One or more gut microbiota GEMs provided by the user, loadable by cobrapy
- Selection of microbes whose fluxes will be incorporated
- Weighting factor α (e.g. 0.5) balancing original lower bounds against aggregated microbial fluxes

## Outputs

- recon3d_modified.xml — SBML-format modified human GEM with exchange-reaction lower bounds reflecting the aggregated microbial in- and out-fluxes, loadable by cobrapy

## How to apply

Load the human GEM (e.g. RECON3D) into cobrapy in Python, then load at least one user-provided gut microbiota GEM and select the microbes whose fluxes will be incorporated. Execute the M2R five-step pipeline end-to-end, aggregating the in- and out-fluxes from the selected microbes; the key parameter is the weighting factor α balancing the original reaction lower bounds against the aggregated microbial in- and out-fluxes (the paper's demonstration uses α = 0.5). M2R modifies the lower bounds of exchange reactions in the human model using these aggregated fluxes, then exports the modified GEM as recon3d_modified.xml in SBML format loadable by cobrapy. Rationale: modifying exchange lower bounds is the mechanism by which the pool of metabolites that enter and leave the human GEM is changed, capturing microbiota–host metabolic interaction.

## Related tools

- **M2R** (Python add-on to cobrapy that aggregates in- and out-fluxes from selected gut microbiota models and modifies the lower bounds of exchange reactions in the human GEM) — https://github.com/e-weglarz-tomczak/m2r
- **cobrapy** (Loads RECON3D and the microbiota GEMs, hosts the M2R add-on, and re-loads the exported SBML for validation)
- **Python** (Runtime environment in which the M2R five-step pipeline is executed end-to-end)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('recon3d_modified.xml')
```

## Evaluation signals

- recon3d_modified.xml exists after the pipeline runs.
- recon3d_modified.xml loads in cobrapy without errors.
- Exchange-reaction lower bounds in the modified model reflect the aggregated in- and out-fluxes from the selected gut microbiota models.
- The α weighting (0.5 in the demonstration) is applied as intended when balancing original lower bounds against microbial fluxes.
- Biological plausibility of the modified metabolite pool is reviewed by an expert.

## Limitations

- Further biological plausibility of the modified metabolite pool is subject to expert review — the workflow only validates structural/syntactic correctness.
- Requires gut microbiota GEMs to be provided by the user; M2R only incorporates fluxes from the selected microbes.
- No changelog found for the software, complicating version tracking.

## Evidence

- [abstract] M2R mechanism: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [abstract] M2R purpose and tooling: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] M2R availability: "M2R is freely available under an MIT License at https://github.com/e-weglarz-tomczak/m2r."
