---
name: cross-library-pipeline-portability-testing
description: Use when a matchms library-cleaning workflow (fixed filter order + settings
  captured in YAML) must be validated as a portable configuration on a different public
  spectral library, e.g., porting the GNPS setup to MassBank or MoNA.
when_to_use_negative:
- Input is a raw LC-MS/MS sample dataset needing peak picking rather than an existing
  annotated spectral library — that is handled by tools like MZMine, MSDial, or OpenMS,
  not this skill
- The goal is to validate chemical annotations against measured fragments — the article
  states such fragment-matching plausibility checks are not yet implemented and are
  only a possible future expansion
- Only a one-off cleaning of a single library is needed with no intent to replicate
  the configuration elsewhere — then a simpler pipeline configuration without the
  portability/reproducibility framing suffices
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the SpectrumProcessor/Pipeline class used to run the fixed library-cleaning
    filter order on each library and to auto-generate the YAML workflow file and processing
    report; version 0.26.4 was used
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and cross-compares SMILES, InChI and InChIKey annotations during the
    Require Valid Annotation filter, so annotation consistency is validated identically
    across libraries
- name: Python
  role: Runtime environment for importing, processing, cleaning, and comparing mass
    spectrometry data via the matchms package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:f4b9c24bf2d678f32e212f573faaa918312d3afc1332391a43b25c922d55157e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-library-pipeline-portability-testing

## Summary

Tests whether a fixed matchms library-cleaning pipeline configuration, captured in a shareable YAML file, can be reused unchanged to clean other public mass spectral libraries beyond the one it was developed on (GNPS), here MassBank and MoNA.

## When to use

Use when a matchms library-cleaning workflow (fixed filter order + settings captured in YAML) must be validated as a portable configuration on a different public spectral library, e.g., porting the GNPS setup to MassBank or MoNA.

## When NOT to use

- Input is a raw LC-MS/MS sample dataset needing peak picking rather than an existing annotated spectral library — that is handled by tools like MZMine, MSDial, or OpenMS, not this skill
- The goal is to validate chemical annotations against measured fragments — the article states such fragment-matching plausibility checks are not yet implemented and are only a possible future expansion
- Only a one-off cleaning of a single library is needed with no intent to replicate the configuration elsewhere — then a simpler pipeline configuration without the portability/reproducibility framing suffices

## Inputs

- Public mass spectral library files in native formats (MassBank records, MoNA/MassBank of North America exports)
- YAML workflow file capturing the fixed matchms filter order and settings
- matchms 0.26.4 environment with Python and RDKit installed

## Outputs

- Cleaned library file per source library
- Automatically generated processing report per run, with per-filter counts of removed spectra, changed metadata, and changed peaks
- YAML workflow file capturing all settings and filters for reproducibility

## How to apply

Download the target libraries (MassBank and MoNA) in their native metadata styles and file formats, then configure matchms 0.26.4 (SpectrumProcessor/Pipeline) with the default library-cleaning filter set so the fixed global filter order is applied unchanged. Run the identical configuration on each library: importing spectra, harmonizing metadata, and applying annotation-repair filters such as Derive Annotation From Compound Name (PubChem-derived SMILES/InChI/InChIKey), Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, and Require Valid Annotation (RDKit-based comparison). For each run, confirm that both a cleaned library file and an automatically generated processing report are produced, and that the report enumerates the filters in the fixed global order with per-filter spectrum-change counts. Compare these counts against the supplementary report tables (Tables S3–S5) to judge that the ported run behaved consistently. The YAML workflow file auto-created by the pipeline is the artifact that guarantees the identical cleaning run is replicable on any other public spectral library.

## Related tools

- **matchms** (Provides the SpectrumProcessor/Pipeline class used to run the fixed library-cleaning filter order on each library and to auto-generate the YAML workflow file and processing report; version 0.26.4 was used) — https://github.com/matchms/matchms
- **RDKit** (Loads and cross-compares SMILES, InChI and InChIKey annotations during the Require Valid Annotation filter, so annotation consistency is validated identically across libraries)
- **Python** (Runtime environment for importing, processing, cleaning, and comparing mass spectrometry data via the matchms package)

## Examples

```
from matchms import Pipeline; pipeline = Pipeline("library_cleaning_workflow.yaml"); pipeline.run("MassBank.msp", "MassBank_cleaned.msp")
```

## Evaluation signals

- Each run produces BOTH a cleaned library file and a processing report; absence of either indicates the ported run failed
- The processing report enumerates the filters applied in the fixed global order, not a re-ordered or subset set of filters
- Per-filter spectrum-change counts in each report are comparable to the supplementary report tables (Tables S3–S5) for the reference runs
- The auto-created YAML file from each run matches the intended fixed configuration, allowing exact diff-based verification of pipeline portability
- Spectra removed by Require Valid Annotation correspond to incomplete or mutually inconsistent SMILES/InChI/InChIKey annotations as loaded by RDKit

## Limitations

- Current filters lack plausibility checks considering both metadata and measured fragments; wrong chemical annotations consistent with the measured mass go unnoticed
- Metadata fields such as instrument type and collision energies are not yet standardized by the current matchms filters
- Runtime is substantial: running the pipeline on the GNPS library of 500,569 spectra took 6 h and 45 min, so ported runs on large libraries need comparable compute
- Deriving SMILES from compound name depends on PubChem matching the given parent mass, so libraries with poor compound-name coverage may lose more annotations

## Evidence

- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] The matchms pipeline was also run on the Massbank library [15]: "The matchms pipeline was also run on the Massbank library [15]"
- [methods] YAML file creation: "it automatically creates a YAML file containing all settings and filters used."
- [methods] processing report content: "This report gives an overview of the number of spectra that were changed in the library by each filter."
- [readme] Pipeline class and yaml workflows: [evidence span withheld — anchor status: external_doc]
- [discussion] no fragment plausibility checks: "they currently still lack plausibility checks that consider both metadata and measured fragments."
- [discussion] unstandardized metadata fields: "the instrument type or collision energies could be important fields that might require additional standardization"
