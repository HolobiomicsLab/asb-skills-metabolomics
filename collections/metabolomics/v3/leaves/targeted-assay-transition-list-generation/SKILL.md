---
name: targeted-assay-transition-list-generation
description: Use when a targeted PRM/MRM lipidomics assay must be built from a lipid
  knowledge base and you need precursor m/z, fragment m/z, adduct, and charge-state
  transitions computed in silico from per-lipid-class rules.
when_to_use_negative:
- The analysis goal is untargeted/discovery lipidomics screening rather than targeting
  a defined subset of lipids of interest — the authors note a strong need to 'target
  distinct sets of molecules in line with their associated biological function, in
  contrast to screening approaches'
- You need automated false discovery rate scoring or resolution of coeluting/isobaric
  lipids and peak area integration — LipidCreator intentionally does not provide a
  dedicated FDR scoring algorithm
- Your target lipidome is bacterial (e.g. E. coli acyl-PG, acyl-PE) or archaeal —
  standards are lacking and the implementation focuses on eukaryotic model systems
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: LipidCreator
  role: Knowledge-based workbench that stores per-class precursor and fragment rules,
    assembles lipid building blocks into precursor/fragment masses, and emits targeted
    transition lists for PRM/MRM experiments
  repo: https://github.com/lifs-tools/lipidcreator/releases
- name: Skyline
  role: Integration target for the generated transition lists and fragment libraries;
    used for small-molecule targeted assay development and DIA method preparation
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:45071c3a993be7ee23eb29d30b93fb5f9d13d8cab078c838b587df19d5f60ac0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - Skyline
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# targeted-assay-transition-list-generation

## Summary

Generates targeted lipidomics assay transition lists by combinatorially assembling lipid head groups, fatty acyl chains/long chain bases, and adducts into precursor and MS/MS fragment masses, following LipidCreator's per-class rule-based computation covering over 61 lipid classes and ~10^12 lipid molecules.

## When to use

Use when a targeted PRM/MRM lipidomics assay must be built from a lipid knowledge base and you need precursor m/z, fragment m/z, adduct, and charge-state transitions computed in silico from per-lipid-class rules.

## When NOT to use

- The analysis goal is untargeted/discovery lipidomics screening rather than targeting a defined subset of lipids of interest — the authors note a strong need to 'target distinct sets of molecules in line with their associated biological function, in contrast to screening approaches'
- You need automated false discovery rate scoring or resolution of coeluting/isobaric lipids and peak area integration — LipidCreator intentionally does not provide a dedicated FDR scoring algorithm
- Your target lipidome is bacterial (e.g. E. coli acyl-PG, acyl-PE) or archaeal — standards are lacking and the implementation focuses on eukaryotic model systems

## Inputs

- LipidCreator knowledge-base tables of lipid classes, lipid molecules, head groups, fatty acyl chains/long chain bases, and allowed adducts
- User-defined lipid species specifications (head group, adduct, fatty acyl or long chain base composition)
- Instrument/polarity configuration (e.g. Thermo QExactive HF PRM, QTRAP 6500 SRM, Agilent QTOF)

## Outputs

- transition_list.csv with precursor m/z, fragment m/z, charge state, polarity, lipid class, and molecule per transition
- Skyline blib spectral library file (after collision energy module activation)
- coverage_summary.csv with per-class molecule tallies and match flags against reported coverage

## How to apply

Load the typed lipid-entity inventory (lipid classes, molecules, head groups, fatty acyls/long chain bases, adducts) from the LipidCreator knowledge-base tables. For each lipid class, apply the precursor-level rules (lipid category, class name, chain length and type, allowed adducts) to combinatorially assemble head group + fatty acyl chains + adduct and compute the monoisotopic precursor m/z from elemental compositions and adduct charge rules. Then apply the class-specific fragment-level calculation rules to enumerate diagnostic MS/MS fragment ions in both positive and negative polarity; LipidCreator's framework computes MS/MS fragment masses for over 60 lipid classes and additionally supports stable isotope labeling, collision energy optimization, and in silico spectral library generation. Emit the transitions (name, mass, adduct, charge state, lipid class, molecule) as a transition list, optionally exported as a Skyline-compatible blib spectral library after activating the collision energy module in the lipid review dialog. Validate by aggregating per-class molecule counts against the reported coverage (10^12 lipid molecules, 61+ classes) and checking that every enumerated molecule has at least one precursor/fragment pair.

## Related tools

- **LipidCreator** (Knowledge-based workbench that stores per-class precursor and fragment rules, assembles lipid building blocks into precursor/fragment masses, and emits targeted transition lists for PRM/MRM experiments) — https://github.com/lifs-tools/lipidcreator/releases
- **Skyline** (Integration target for the generated transition lists and fragment libraries; used for small-molecule targeted assay development and DIA method preparation) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- coverage_summary.csv reproduces the reported scale: over 10^12 lipid molecules and over 60 (61+) lipid classes
- transition_list.csv contains at least one precursor/fragment pair for every enumerated lipid molecule
- Every exported transition row includes name, mass, adduct, and charge state fields
- Fragment masses for lipid mediator species match entries validated against the Metlin database
- The blib spectral library export succeeds from the lipid review dialog after activating the collision energy module and adding transitions

## Limitations

- Collision energy models are currently exclusive to lipid mediators; extending them to other lipid classes requires more structurally diverse standards and CE-dependent fragmentation measurements
- Implementation focuses on eukaryotic model systems due to lack of standards for bacteria such as E. coli and archaea
- No dedicated scoring algorithm for false discovery rates; coeluting/isobaric lipids and peak area integration are intentionally not addressed
- Retention time, elution order, and isotope labeled standards must be considered to avoid mis-annotations; no large-scale methods exist for stereoisomers due to lack of isotope-coded standards

## Evidence

- [abstract] Knowledge base coverage: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [abstract] Functionalities: "LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries."
- [other] Per-class rules: [evidence span withheld — anchor status: not_found]
- [discussion] Skyline integration: "LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics."
- [readme] Blib export: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions."
- [discussion] Scope limitation: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
