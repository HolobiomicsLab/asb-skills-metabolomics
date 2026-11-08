---
name: adduct-assembly-combinatorics
description: Use when building targeted lipidomics precursor/fragment transition lists
  from lipid class definitions and adduct rules, or when auditing how many lipid classes/molecules
  an in-silico lipid knowledge base covers.
when_to_use_negative:
- The task needs false-discovery-rate scoring or resolution of coeluting/isobaric
  lipids and peak area integration — LipidCreator intentionally does not address these
- Target lipids are bacterial/archaeal classes such as acyl-PG or acyl-PE in E. coli
  — no standards are available and the knowledge base focuses on eukaryotic model
  systems
- Distinguishing stereoisomers at large scale — no isotope-coded standards exist for
  this in the current implementation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: LipidCreator
  role: Source of the per-class precursor and fragment calculation rules and the lipid
    knowledge base being reconstructed; provides transition-list and blib spectral
    library export
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Consumes the generated target lists and fragment libraries for PRM and MRM
    targeted lipidomics experiments
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:359299429988362f8bbcd74dc6c83b7fb4e7df8b26792af6984212983cb4e173
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# adduct-assembly-combinatorics

## Summary

Reconstructs LipidCreator's combinatorial assembly of lipid head groups, fatty acyl chains (or long chain bases), and adducts into precursor and fragment m/z values, emitting a targeted-assay transition list covering 61+ lipid classes and ~10^12 lipid molecules.

## When to use

Use when building targeted lipidomics precursor/fragment transition lists from lipid class definitions and adduct rules, or when auditing how many lipid classes/molecules an in-silico lipid knowledge base covers.

## When NOT to use

- The task needs false-discovery-rate scoring or resolution of coeluting/isobaric lipids and peak area integration — LipidCreator intentionally does not address these
- Target lipids are bacterial/archaeal classes such as acyl-PG or acyl-PE in E. coli — no standards are available and the knowledge base focuses on eukaryotic model systems
- Distinguishing stereoisomers at large scale — no isotope-coded standards exist for this in the current implementation

## Inputs

- LipidCreator knowledge-base tables of lipid classes, molecules, head groups, fatty acyls, and adducts (from github.com/lifs-tools/lipidcreator releases)
- Per-class precursor-level rules (lipid category, class name, fatty acyl chain length and type, allowed adducts)
- Per-class fragment calculation rules and MS/MS fragment definitions

## Outputs

- transition_list.csv: precursor m/z, fragment m/z, adduct, charge state, polarity, lipid class, molecule name
- coverage_summary.csv: per-class molecule tallies with match flags against the reported >61 classes / 10^12 molecules

## How to apply

Load the typed lipid-entity inventory (lipid categories, class names, fatty acyl chain lengths and types, adducts, head groups, long chain bases) from the LipidCreator knowledge-base tables. Per lipid class, combinatorially assemble head group + fatty acyl chains + adduct and compute monoisotopic precursor m/z from elemental compositions and per-class adduct rules, mirroring how LipidCreator stores rules at the precursor level before they feed the fragment level. Then apply each class-specific fragment calculation rule to enumerate diagnostic MS/MS fragment ions in positive and negative polarity. Export the transition list (precursor m/z, fragment m/z, charge, polarity, lipid class, molecule) and validate coverage against the paper's reported >61 classes and 10^12 molecule array.

## Related tools

- **LipidCreator** (Source of the per-class precursor and fragment calculation rules and the lipid knowledge base being reconstructed; provides transition-list and blib spectral library export) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Consumes the generated target lists and fragment libraries for PRM and MRM targeted lipidomics experiments) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
mono LipidCreator.exe  # stand-alone/command-line operation to assemble precursors/fragments and export transition lists and blib libraries
```

## Evaluation signals

- coverage_summary.csv reproduces the reported scale of >61 lipid classes and the array of 10^12 lipid molecules
- transition_list.csv contains at least one precursor/fragment pair per enumerated molecule
- Every emitted transition row carries a name, mass, adduct, and charge state as in LipidCreator's exported transition lists
- Computed precursor m/z values match LipidCreator outputs for the same class/head group/fatty acyl/adduct combination in positive and negative polarity
- blib spectral library export (after activating the collision energy module) succeeds from the lipid review dialog, validating transition integrity

## Limitations

- Collision energy models are currently exclusive to lipid mediators; extending to other lipid classes requires more structurally diverse lipid standards and CE-dependent measurements
- The majority but not all eukaryotic lipid classes are covered (exceptions in Supplementary Data 1); rare bacterial/archaeal classes and acyl-PG/acyl-PE in E. coli lack standards
- No dedicated scoring algorithm for false discovery rates; retention time, elution order, and isotope-labeled standards must be considered to avoid mis-annotations
- LipidCreator UI relies on .NET 4.5.2 / Mono; under Mono on Linux some scrollable window controls repaint imperfectly, and blib export may require building a custom libSQLite.Interop.so

## Evidence

- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [abstract] LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries.: "LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries."
- [intro] we here present a targeted lipidomics workbench and lipid knowledge-base that is fully integrated with Skyline: "we here present a targeted lipidomics workbench and lipid knowledge-base that is fully integrated with Skyline"
- [discussion] LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics.: "LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics."
- [readme] It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline.: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
- [readme] Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
