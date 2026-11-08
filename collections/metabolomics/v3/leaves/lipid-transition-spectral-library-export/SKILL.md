---
name: lipid-transition-spectral-library-export
description: Use when a targeted lipidomics workflow needs a Skyline-consumable spectral
  library from computed lipid transitions — e.g., exporting CE-optimized platelet
  lipid-mediator transition lists as .blib for spectral matching.
when_to_use_negative:
- The task requires false-discovery-rate scoring, handling coeluting/isobaric lipids,
  or peak area integration — LipidCreator intentionally does not address these.
- The goal is untargeted lipid screening rather than targeted assays on a defined
  set of lipids of interest.
- Working with lipid classes outside the covered eukaryotic set (e.g., acyl-PG/acyl-PE
  in E. coli or archaeal lipids) where standards are lacking.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidCreator
  role: Generates the transition list and exports the Bibliospec .blib spectral library
    after activating the collision energy module
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Consumes the exported .blib for spectral library matching and DIA method preparation
    in targeted lipidomics assays
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: sqlite3
  role: sqlite package required on Linux/Ubuntu for LipidCreator's blib (sqlite-backed)
    spectral library export
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:0e99ff281961e7097772d0e05c2b7eedde6462fd33bbf62a996e6d5916c2bc97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  - sqlite3
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lipid-transition-spectral-library-export

## Summary

Generates a Bibliospec (.blib) in silico spectral library from LipidCreator's lipid review dialog with collision-energy-optimized transitions, for direct consumption by Skyline to validate fragment ion traces in targeted lipidomics (PRM/MRM/DIA) assays.

## When to use

Use when a targeted lipidomics workflow needs a Skyline-consumable spectral library from computed lipid transitions — e.g., exporting CE-optimized platelet lipid-mediator transition lists as .blib for spectral matching.

## When NOT to use

- The task requires false-discovery-rate scoring, handling coeluting/isobaric lipids, or peak area integration — LipidCreator intentionally does not address these.
- The goal is untargeted lipid screening rather than targeted assays on a defined set of lipids of interest.
- Working with lipid classes outside the covered eukaryotic set (e.g., acyl-PG/acyl-PE in E. coli or archaeal lipids) where standards are lacking.

## Inputs

- CE-optimized transition list for target lipids (e.g., platelet lipid mediators) from the LipidCreator lipid review dialog
- Instrument-specific collision energy configuration (minimum CE threshold, e.g., for Thermo QExactive HF or Agilent QTOF)

## Outputs

- Bibliospec .blib spectral library file (sqlite-backed) loadable by Skyline
- Skyline document configured for DIA/spectral-matching method preparation

## How to apply

Load the transition list (e.g., CE-optimized targets for platelet lipid mediators) into LipidCreator's lipid review dialog and add the desired transitions. Activate the collision energy module so each transition carries its model-based CE, respecting the instrument-specific minimum collision energy configuration above which predicted values are reported. Export the transitions as a Bibliospec .blib spectral library; on Linux/Ubuntu this requires the sqlite3 package (`sudo apt install sqlite3`) since the blib format is sqlite-backed. If the precompiled libSQLite.Interop.so fails on the target distribution, rebuild it from sqlite-netFx-source-1.0.111.0 via `bash compile-interop-assembly-release.sh` in the Setup folder and copy the resulting library next to LipidCreator.exe. Validate by loading the .blib into Skyline and confirming it supports the DIA/spectral-matching method preparation (e.g., reproducing the transition set from the platelet mediator assay on the QEx HF / QTRAP 6500).

## Related tools

- **LipidCreator** (Generates the transition list and exports the Bibliospec .blib spectral library after activating the collision energy module) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Consumes the exported .blib for spectral library matching and DIA method preparation in targeted lipidomics assays) — https://skyline.ms/project/home/software/Skyline/begin.view
- **sqlite3** (sqlite package required on Linux/Ubuntu for LipidCreator's blib (sqlite-backed) spectral library export)

## Examples

```
sudo apt install sqlite3 && mono LipidCreator.exe  # then export the .blib from the lipid review dialog with the CE module active
```

## Evaluation signals

- The exported lipid_mediator_assay.blib loads into Skyline without error.
- The .blib reproduces the exact transition set (with model-based collision energies) used in the reported platelet lipid-mediator validation.
- Each transition in the library carries a CE value at or above the instrument-specific minimum CE, confirming the CE module was active during export.
- On Linux, the blib export completes without the libSQLite.Interop.so failure; otherwise the rebuilt interop library (sqlite-netFx-source-1.0.111.0) was placed beside LipidCreator.exe.

## Limitations

- CE models are currently exclusive to lipid mediators; extending to other lipid classes requires more structurally diverse lipid standards and CE-dependent fragmentation measurements.
- On Linux/Ubuntu, Mono may cause repainting issues in scrollable windows/controls; the precompiled libSQLite.Interop.so may fail on non-Debian distributions and must be rebuilt from source.
- LipidCreator provides no dedicated FDR scoring and does not address coeluting/isobaric lipids or peak area integration; retention time, elution order, and isotope-labeled standards must be considered to avoid mis-annotations.

## Evidence

- [methods] Export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [readme] In order to use the blib spectral library export functionality, please install the appropriate sqlite (>3) package for your distribution.: "In order to use the blib spectral library export functionality, please install the appropriate sqlite (>3) package for your distribution."
- [discussion] the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values.: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [discussion] DIA method preparation and data analysis were performed with Skyline: "DIA method preparation and data analysis were performed with Skyline (See Supplementary Methods)."
- [abstract] LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries.: "LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries."
