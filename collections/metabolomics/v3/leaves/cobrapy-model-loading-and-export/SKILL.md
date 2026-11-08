---
name: cobrapy-model-loading-and-export
description: Use when an analysis requires reading a genome-scale metabolic model
  (e.g. RECON3D SBML) or microbiota GEMs into cobrapy, or writing a modified human
  GEM back to SBML for downstream FBA or M2R processing.
when_to_use_negative:
- The task is to compute or aggregate the microbial in- and out-fluxes themselves
  — that is M2R's five-step modification pipeline, not plain model I/O.
- The input is not a metabolic reconstruction (e.g. raw flux data, abundance tables,
  or non-SBML model formats cobrapy cannot read).
- The target model is not a human/microbiota GEM compatible with RECON3D-style exchange
  reactions.
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: cobrapy
  role: Python library used to load SBML GEMs into memory (read_sbml_model) and export
    modified human GEMs back to SBML (write_sbml_model); M2R runs as an add-on on
    top of it.
  repo: https://github.com/e-weglarz-tomczak/m2r
- name: M2R
  role: Add-on that consumes the cobrapy-loaded models and produces the modified GEM
    whose lower bounds have been updated from aggregated microbial fluxes before export.
  repo: https://github.com/e-weglarz-tomczak/m2r
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:a522e4d5bd4c297b74c989d6d95abb3c90c09778502b915424d50c88ded499b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/e-weglarz-tomczak/m2r
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cobrapy-model-loading-and-export

## Summary

Load human genome-scale metabolic models such as RECON3D and gut microbiota GEMs into cobrapy (Python) as COBRApy Model objects, and export modified models back out as SBML XML files (e.g. recon3d_modified.xml). This is the foundational I/O step that makes M2R's exchange-reaction lower-bound modification possible.

## When to use

Use when an analysis requires reading a genome-scale metabolic model (e.g. RECON3D SBML) or microbiota GEMs into cobrapy, or writing a modified human GEM back to SBML for downstream FBA or M2R processing.

## When NOT to use

- The task is to compute or aggregate the microbial in- and out-fluxes themselves — that is M2R's five-step modification pipeline, not plain model I/O.
- The input is not a metabolic reconstruction (e.g. raw flux data, abundance tables, or non-SBML model formats cobrapy cannot read).
- The target model is not a human/microbiota GEM compatible with RECON3D-style exchange reactions.

## Inputs

- RECON3D human genome-scale metabolic model (SBML .xml file loadable by cobrapy)
- One or more user-provided gut microbiota GEMs (SBML files loadable by cobrapy)

## Outputs

- In-memory cobrapy Model objects for the human GEM and each microbiota GEM
- Modified human GEM exported as recon3d_modified.xml (SBML format loadable by cobrapy)

## How to apply

Load the RECON3D human GEM into cobrapy in Python using cobra.io.read_sbml_model, and load each user-provided gut microbiota GEM the same way before selecting which microbes' fluxes will be incorporated. Keep each model as a separate in-memory COBRApy Model so exchange reactions (the boundary reactions whose lower bounds M2R modifies) remain identifiable by their standard exchange-reaction identifiers. After M2R's five-step pipeline aggregates the microbial in- and out-fluxes (with weighting factor alpha, e.g. 0.5 in the paper's demonstration) and applies them to the exchange-reaction lower bounds, export the modified human GEM with cobra.io.write_sbml_model to recon3d_modified.xml in SBML format. The rationale is that SBML is the interchange format both cobrapy and M2R operate on, so round-tripping through a single SBML file preserves model structure and bounds for reuse or expert review.

## Related tools

- **cobrapy** (Python library used to load SBML GEMs into memory (read_sbml_model) and export modified human GEMs back to SBML (write_sbml_model); M2R runs as an add-on on top of it.) — https://github.com/e-weglarz-tomczak/m2r
- **M2R** (Add-on that consumes the cobrapy-loaded models and produces the modified GEM whose lower bounds have been updated from aggregated microbial fluxes before export.) — https://github.com/e-weglarz-tomczak/m2r

## Examples

```
import cobra; model = cobra.io.read_sbml_model('recon3d.xml'); microbe = cobra.io.read_sbml_model('microbiota_gem.xml'); cobra.io.write_sbml_model(model, 'recon3d_modified.xml')
```

## Evaluation signals

- recon3d_modified.xml exists on disk after the workflow completes.
- recon3d_modified.xml loads in cobrapy via read_sbml_model without errors.
- The exchange-reaction lower bounds in the exported model reflect the aggregated in- and out-fluxes from the selected gut microbiota models (with weighting factor alpha = 0.5 in the demonstration).
- The original RECON3D file is unchanged and the modified file can be re-loaded and re-optimized without model-structure corruption.

## Limitations

- M2R only modifies the lower bounds of exchange reactions; any further biological plausibility of the modified metabolite pool is subject to expert review.
- No changelog was found for the M2R repository, so version-specific I/O behavior must be verified against the installed release.

## Evidence

- [abstract] cobrapy as loader/exporter of SBML GEMs: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] lower bounds of exchange reactions are the modified quantities: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [other] export target recon3d_modified.xml: [evidence span withheld — anchor status: not_found]
