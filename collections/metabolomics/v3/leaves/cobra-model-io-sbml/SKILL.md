---
name: cobra-model-io-sbml
description: Use when a task requires reading a human GEM such as RECON3D or a gut
  microbiota metabolic model into cobrapy, or saving a bounds-modified GEM (e.g. as
  recon3d_modified.xml) for downstream flux analysis.
when_to_use_negative:
- The input model is already an in-memory cobra.Model object and no file round-trip
  is needed — skip the SBML I/O and apply the bound updates directly.
- The goal is to modify upper bounds or stoichiometry rather than lower bounds of
  exchange reactions, which is outside M2R's described scope.
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_2250
tools:
- name: cobrapy
  role: Provides the SBML read/write functions used to load RECON3D and the microbiota
    model and to save the modified GEM; M2R is a Python add-on to cobrapy
  repo: github.com/e-weglarz-tomczak/m2r
- name: M2R
  role: Modifies the lower bounds of exchange reactions in the loaded GEM using aggregated
    in- and out-fluxes from selected microbes before the model is written back to
    SBML
  repo: github.com/e-weglarz-tomczak/m2r
- name: Python
  role: Execution environment in which the cobrapy model I/O and M2R bound-update
    steps are scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab060
    title: 'M2R: a Python add-on to cobrapy for modifying human genome-scale metabolic
      reconstruction using the gut microbiota models'
  source_content_hash: sha256:1ac62cd7976378ec897309e96f3e6403c0b45dc701088bb0be73275b54a8f493
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - M2R
  - Python
  - cobrapy
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# cobra-model-io-sbml

## Summary

Load and write genome-scale metabolic models (GEMs) in SBML XML format using cobrapy's I/O functions, as the entry and exit points for workflows like M2R that modify human reconstructions (e.g. RECON3D) with gut microbiota flux information.

## When to use

Use when a task requires reading a human GEM such as RECON3D or a gut microbiota metabolic model into cobrapy, or saving a bounds-modified GEM (e.g. as recon3d_modified.xml) for downstream flux analysis.

## When NOT to use

- The input model is already an in-memory cobra.Model object and no file round-trip is needed — skip the SBML I/O and apply the bound updates directly.
- The goal is to modify upper bounds or stoichiometry rather than lower bounds of exchange reactions, which is outside M2R's described scope.

## Inputs

- Human genome-scale metabolic model in SBML XML format (e.g. RECON3D)
- Gut microbiota metabolic model in SBML XML format
- User-specified weighting parameter α ∈ [0,1] and aggregated microbiota in/out-fluxes (fi, fo)

## Outputs

- cobra.Model objects loaded in memory from the SBML XML files
- Modified host GEM written as SBML XML (e.g. recon3d_modified.xml)

## How to apply

Call cobrapy's SBML reader to parse the host GEM (RECON3D) and the gut microbiota metabolic model from XML into cobra.Model objects in Python, then scan the host model for reactions containing metabolites whose identifiers are present in the microbiota model. After M2R applies its lower-bound update LBr := α·LBr + (1−α)·fi − (1−α)·fo (α ∈ [0,1]; α = 0.5 in the paper's demonstration, fi/fo the aggregated in- and out-fluxes from selected microbes), use cobrapy's SBML writer to save the modified model, e.g. as recon3d_modified.xml. Because SBML round-trips can drop or rename annotation, re-read the written file and recompute the updated bounds from the stored original lower bounds and flux aggregates to confirm no off-target reactions changed.

## Related tools

- **cobrapy** (Provides the SBML read/write functions used to load RECON3D and the microbiota model and to save the modified GEM; M2R is a Python add-on to cobrapy) — github.com/e-weglarz-tomczak/m2r
- **M2R** (Modifies the lower bounds of exchange reactions in the loaded GEM using aggregated in- and out-fluxes from selected microbes before the model is written back to SBML) — github.com/e-weglarz-tomczak/m2r
- **Python** (Execution environment in which the cobrapy model I/O and M2R bound-update steps are scripted)

## Examples

```
python -c "import cobra; model = cobra.io.read_sbml_model('RECON3D.xml'); cobra.io.write_sbml_model(model, 'recon3d_modified.xml')"
```

## Evaluation signals

- Re-read the written SBML (recon3d_modified.xml) and confirm every updated bound equals α·LBr + (1−α)·fi − (1−α)·fo exactly.
- Verify that reactions whose metabolites match no microbiota metabolite retain their original lower bounds (no off-target changes).
- Confirm only reactions containing at least one metabolite whose identifier is present in the microbiota model were modified.
- Check that non-exchange reactions and model stoichiometry survive the SBML round-trip without alteration.
- Confirm the saved file is a valid SBML XML that cobrapy can re-load as a cobra.Model without errors.

## Limitations

- M2R only modifies the lower bounds of exchange reactions via aggregated in- and out-fluxes from selected microbes; other model aspects are untouched.
- The update depends on a user-chosen α ∈ [0,1] (0.5 in the demonstration); a poor α choice can over- or under-weight the original bound relative to microbiota fluxes.
- SBML round-tripping through cobrapy may not preserve all model annotations, so validation after writing is necessary.

## Evidence

- [abstract] M2R as Python add-on to cobrapy for RECON3D GEMs: "Here, we present M2R, a Python add-on to cobrapy that allows incorporating information about the gut microbiota metabolism models to human genome-scale metabolic models (GEMs) like RECON3D."
- [abstract] Lower-bound modification mechanism: "The idea behind the software is to modify the lower bounds of the exchange reactions in the model using aggregated in- and out-fluxes from selected microbes."
- [other] Update rule and output file from task workflow: [evidence span withheld — anchor status: not_found]
