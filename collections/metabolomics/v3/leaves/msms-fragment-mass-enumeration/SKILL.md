---
name: msms-fragment-mass-enumeration
description: Use when building targeted PRM/MRM or DIA assays for lipids and you must
  enumerate precursor-to-fragment transitions (m/z, adduct, charge) for lipid classes
  rather than measuring them experimentally.
when_to_use_negative:
- The goal is untargeted lipid discovery or de novo identification — LipidCreator
  intentionally does not address coeluting/isobaric lipids, peak area integration,
  or false discovery rate scoring, offering only knowledge-based transition list selection.
- Target lipids are from bacteria such as E. coli (acyl-PG, acyl-PE) or archaea, where
  explicit lack of standards means the eukaryotic-focused knowledge base is not covered.
- The task requires stereoisomer-resolved quantification — no large-scale methods
  exist to cope with stereoisomers due to lack of isotope-coded standards.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipidCreator
  role: Performs the precursor and MS/MS fragment mass computation from its lipid
    knowledge-base; standalone, command-line, or as a Skyline plugin
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Consumes the generated target lists and fragment libraries for PRM/MRM/DIA
    method preparation and blib spectral library export
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:d2c4da3deb720f3939b7edcfa329443898b20f2e260cd8ed9a58c743b7fce913
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msms-fragment-mass-enumeration

## Summary

Enumerate targeted lipidomics precursor and MS/MS fragment transition lists by assembling head groups, fatty acyl chains/long chain bases, and adducts per lipid class, computing monoisotopic masses and class-specific fragment ions — the core computation performed by LipidCreator's knowledge-base for over 61 lipid classes spanning 10^12 lipid molecules.

## When to use

Use when building targeted PRM/MRM or DIA assays for lipids and you must enumerate precursor-to-fragment transitions (m/z, adduct, charge) for lipid classes rather than measuring them experimentally.

## When NOT to use

- The goal is untargeted lipid discovery or de novo identification — LipidCreator intentionally does not address coeluting/isobaric lipids, peak area integration, or false discovery rate scoring, offering only knowledge-based transition list selection.
- Target lipids are from bacteria such as E. coli (acyl-PG, acyl-PE) or archaea, where explicit lack of standards means the eukaryotic-focused knowledge base is not covered.
- The task requires stereoisomer-resolved quantification — no large-scale methods exist to cope with stereoisomers due to lack of isotope-coded standards.

## Inputs

- Typed lipid-entity inventory: lipid classes, lipid molecules, head groups, fatty acyl chains, long chain bases, adduct rules (LipidCreator knowledge-base tables)
- Lipid class-specific fragment calculation/definition rules
- Instrument-specific collision energy configuration (minimum CE threshold)
- User-defined target list specification for PRM or MRM experiments

## Outputs

- Transition list (precursor m/z, fragment m/z, name, mass, adduct, charge state, polarity, lipid class, molecule)
- In silico spectral library (blib file via Skyline/blib export)
- Per-class coverage tallies (61+ lipid classes, 10^12 lipid molecules)
- Target lists and fragment libraries for Skyline PRM/MRM

## How to apply

Load the typed lipid-entity inventory (lipid category, class name, fatty acyl chain length and type, adducts, head groups) from the knowledge-base tables, then combinatorially assemble head group + fatty acyls (or long chain bases) + adduct per class and compute monoisotopic precursor m/z from elemental compositions and adduct rules. Next, apply class-level fragment calculation rules to enumerate diagnostic MS/MS fragment ions in both positive and negative polarity — multiple fragments and both polarity modes are required for unequivocal identification in unvalidated assays. Emit the transition list with name, mass, adduct, and charge state; within Skyline, transitions can be exported to a blib spectral library after activating the collision energy module, which supports instrument-specific minimum collision energy thresholds above which model-based predicted values are reported. Validate that every enumerated molecule has at least one precursor/fragment pair and that per-class tallies reproduce the reported coverage scale.

## Related tools

- **LipidCreator** (Performs the precursor and MS/MS fragment mass computation from its lipid knowledge-base; standalone, command-line, or as a Skyline plugin) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Consumes the generated target lists and fragment libraries for PRM/MRM/DIA method preparation and blib spectral library export)

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- Coverage summary reproduces the reported scale: over 10^12 lipid molecules and over 60/61 lipid classes.
- transition_list.csv contains at least one precursor/fragment pair per enumerated molecule, with name, mass, adduct, and charge state fields populated.
- Fragment masses validate against the Metlin database (https://metlin.scripps.edu/) for lipid mediator species.
- blib export succeeds from the lipid review dialog after activating the collision energy module and adding transitions (requires sqlite3 on Linux).
- Computed assays reproduce experimental results on NIST SRM 1950 standard material and human plasma lipidome samples.

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids and peak area integration.
- Collision energy prediction models are currently exclusive to lipid mediators; expanding to other lipid classes requires more structurally diverse lipid standards and CE-dependent fragmentation measurements.
- Retention time, elution order, and isotope labeled standards must additionally be considered to avoid mis-annotations; no large-scale methods exist for stereoisomers due to lack of isotope-coded standards.
- Current implementation focuses on eukaryotic model systems; rare bacterial (E. coli acyl-PG, acyl-PE) and archaeal lipid classes lack standards.
- On Linux under Mono, repainting of certain windows/scrollable areas may be imperfect, and the native libSQLite.Interop.so may need manual compilation for blib export.

## Evidence

- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [intro] Targeted workbench integrated with Skyline: "we here present a targeted lipidomics workbench and lipid knowledge-base that is fully integrated"
- [discussion] Skyline integration for vendor-independent assay development: "LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics."
- [readme] Creates target lists and fragment libraries for PRM and MRM: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
- [methods] Source releases available from GitHub: "Source releases are available from the [Github release page](https://github.com/lifs-tools/lipidcreator/releases)"
