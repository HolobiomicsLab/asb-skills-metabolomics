---
name: skyline-spectral-library-import-validation
description: Use when targeted lipidomics transitions (e.g. platelet lipid mediators)
  need an in silico .blib spectral library consumable by Skyline for DIA/spectral-matching
  method validation.
when_to_use_negative:
- Input is already a validated .blib library with matching transitions confirmed in
  Skyline.
- Lipid classes outside LipidCreator's CE-validated scope — the CE models are exclusive
  to lipid mediators and expanding requires more structurally diverse lipid standards.
- Workflow needs automated false discovery rate scoring or handling of coeluting/isobaric
  lipids — LipidCreator intentionally does not address these.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: LipidCreator
  role: Generates the transition list and exports it as a Bibliospec .blib spectral
    library after activating the collision energy module in the lipid review dialog.
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Consumes the exported .blib to validate fragment ion traces from targeted
    lipidomics assays via spectral library matching for DIA method preparation.
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: sqlite3
  role: Underlying sqlite dependency enabling .blib export on Linux/Ubuntu; install
    via sudo apt install sqlite3.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:7a74d921db4f3e0028e64288562cfb21e658a5ec4fcf95b83e4fb3606ffcc56c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  - sqlite3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Skyline spectral library import validation

## Summary

Exports LipidCreator transitions as a Bibliospec .blib spectral library and loads it into Skyline to validate that fragment ion traces for targeted lipidomics assays resolve via spectral matching.

## When to use

Use when targeted lipidomics transitions (e.g. platelet lipid mediators) need an in silico .blib spectral library consumable by Skyline for DIA/spectral-matching method validation.

## When NOT to use

- Input is already a validated .blib library with matching transitions confirmed in Skyline.
- Lipid classes outside LipidCreator's CE-validated scope — the CE models are exclusive to lipid mediators and expanding requires more structurally diverse lipid standards.
- Workflow needs automated false discovery rate scoring or handling of coeluting/isobaric lipids — LipidCreator intentionally does not address these.

## Inputs

- Transition list exported from LipidCreator's lipid review dialog (with collision energy module activated)
- Bibliospec .blib spectral library file
- sqlite3 runtime package (Linux/Ubuntu)
- Native libSQLite.Interop.so interop library

## Outputs

- Validated .blib spectral library loadable in Skyline (e.g. lipid_mediator_assay.blib)
- Confirmed fragment ion traces for targeted assays via spectral matching in Skyline

## How to apply

Add transitions in LipidCreator's lipid review dialog, then activate the collision energy module so each transition carries its model-based CE honoring the instrument-specific minimum CE configuration before export. Export the transition list as a Bibliospec .blib file; on Linux/Ubuntu install the sqlite3 package (sudo apt install sqlite3) since the blib export depends on sqlite. If the shipped precompiled libSQLite.Interop.so fails on your distribution, download sqlite-netFx-source-1.0.111.0.zip, run Setup/compile-interop-assembly-release.sh with a GCC toolchain, and copy ../bin/2013/Release/bin/libSQLite.Interop.so next to LipidCreator.exe. Transmit the .blib directly to Skyline and load it to confirm it supports the DIA/spectral-matching method preparation for the target assay (e.g. the platelet mediator experiment), mirroring the article's reported validation.

## Related tools

- **LipidCreator** (Generates the transition list and exports it as a Bibliospec .blib spectral library after activating the collision energy module in the lipid review dialog.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Consumes the exported .blib to validate fragment ion traces from targeted lipidomics assays via spectral library matching for DIA method preparation.) — https://skyline.ms/project/home/software/Skyline/begin.view
- **sqlite3** (Underlying sqlite dependency enabling .blib export on Linux/Ubuntu; install via sudo apt install sqlite3.)

## Examples

```
mono LipidCreator.exe; sudo apt install sqlite3; bash Setup/compile-interop-assembly-release.sh
```

## Evaluation signals

- The exported lipid_mediator_assay.blib loads into Skyline without error.
- The loaded library reproduces the transition set used in the reported platelet mediator validation.
- Each transition in the blib carries a model-based collision energy respecting the instrument-specific minimum CE configuration.
- Blib export succeeds after activating the collision energy module in the lipid review dialog, per the README acceptance test.
- If the precompiled libSQLite.Interop.so was rebuilt, the copy placed beside LipidCreator.exe resolves the prior export failure.

## Limitations

- CE models are currently exclusive to lipid mediators; extending to other lipid classes requires availability of more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior.
- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates, and intentionally does not address coeluting/isobaric lipids and peak area integration.
- Retention time, elution order of lipids, and isotope labeled standards must be considered to avoid mis-annotations.
- Precompiled native libSQLite.Interop.so may fail on non-Debian/Ubuntu Linux distributions, requiring a custom rebuild from the sqlite-netFx source.
- Mono on Linux has imperfect repaint behavior for scrollable windows and controls.

## Evidence

- [methods] blib export workflow in lipid review dialog: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [intro] Skyline integration for targeted assays: "we here present a targeted lipidomics workbench and lipid knowledge-base that is fully integrated with Skyline"
- [abstract] in silico spectral library generation capability: "LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries."
- [readme] sqlite3 dependency for blib export on Linux: "In order to use the blib spectral library export functionality, please install the appropriate sqlite (>3) package for your distribution."
- [readme] Mono repaint limitations on Linux: "Please note that there may be issues with the repainting of certain windows and controls (scrollable areas) due to the not 100% compatible Mono implementation."
