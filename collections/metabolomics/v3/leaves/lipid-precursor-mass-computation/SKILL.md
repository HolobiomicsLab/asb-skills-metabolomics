---
name: lipid-precursor-mass-computation
description: 'Use when designing targeted lipidomics PRM/MRM assays: you need to compute
  monoisotopic precursor m/z and diagnostic fragment masses from lipid class definitions,
  fatty acyl chains, and adducts, or generate transition lists for Skyline.'
when_to_use_negative:
- Your input is already a validated transition list or spectral library — LipidCreator
  does not re-score or re-calculate existing assays.
- You need automated false discovery rate scoring or handling of coeluting/isobaric
  lipids — LipidCreator intentionally does not address this.
- Your targets are bacterial (e.g. E. coli acyl-PG, acyl-PE) or archaeal lipids —
  no standards exist and the current knowledge base is eukaryotic-focused.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: LipidCreator
  role: 'Primary tool: stores per-class precursor and fragment calculation rules in
    its knowledge base, assembles head groups/fatty acyls/adducts into precursor and
    fragment masses, and exports transition lists or blib libraries.'
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: 'Integration host: LipidCreator is a Skyline plugin for creating user-defined
    target lists and fragment libraries for PRM/MRM experiments.'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:5e76d10098d61092c7ea10a4660015d50c2ff4092c29690e355dddc027c15b9d
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
yes_rate: 1.0
n_claims: 8
---

# lipid-precursor-mass-computation

## Summary

Reconstructs LipidCreator's knowledge-base-driven assembly of lipid building blocks (head groups, fatty acyls, adducts) into precursor and fragment masses for over 61 lipid classes, emitting targeted-assay transition lists covering ~10^12 lipid molecules.

## When to use

Use when designing targeted lipidomics PRM/MRM assays: you need to compute monoisotopic precursor m/z and diagnostic fragment masses from lipid class definitions, fatty acyl chains, and adducts, or generate transition lists for Skyline.

## When NOT to use

- Your input is already a validated transition list or spectral library — LipidCreator does not re-score or re-calculate existing assays.
- You need automated false discovery rate scoring or handling of coeluting/isobaric lipids — LipidCreator intentionally does not address this.
- Your targets are bacterial (e.g. E. coli acyl-PG, acyl-PE) or archaeal lipids — no standards exist and the current knowledge base is eukaryotic-focused.

## Inputs

- LipidCreator knowledge-base tables (lipid classes, head groups, fatty acyls, long chain bases, adducts)
- User-defined lipid target list (lipid category, class, fatty acyl chain length/type, adduct, polarity)
- Stable isotope labeling specification

## Outputs

- transition_list.csv (precursor m/z, fragment m/z, charge, polarity, lipid class, molecule name)
- blib spectral library file
- coverage_summary.csv (per-class molecule tallies vs. reported 10^12 molecules / 61+ classes)

## How to apply

Load the typed lipid-entity inventory (lipid categories, class names, head groups, fatty acyl chain lengths/types, long chain bases, adducts) from LipidCreator's knowledge-base tables. Implement per-class combinatorial assembly of head group + fatty acyl chains + adduct, computing monoisotopic precursor m/z from elemental compositions and adduct rules. Apply class-specific fragment definitions to enumerate diagnostic MS/MS fragment ions in positive and negative polarity; the precursor-level rules feed the fragment level whose calculation rules generate fragment masses. Emit the transition list with name, mass, adduct, and charge state as transition_list.csv, and validate that the implemented inventory covers over 60 lipid classes and ~10^12 lipid molecules with at least one precursor/fragment pair per enumerated molecule. Export as a blib spectral library for Skyline when the collision energy module is activated.

## Related tools

- **LipidCreator** (Primary tool: stores per-class precursor and fragment calculation rules in its knowledge base, assembles head groups/fatty acyls/adducts into precursor and fragment masses, and exports transition lists or blib libraries.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Integration host: LipidCreator is a Skyline plugin for creating user-defined target lists and fragment libraries for PRM/MRM experiments.)

## Examples

```
mono LipidCreator.exe --target-list targets.csv --adducts H+,NH4+ --polarity both --export-blib
```

## Evaluation signals

- coverage_summary.csv must show >60 lipid classes and ~10^12 lipid molecules, matching the article's reported scale.
- transition_list.csv must contain at least one precursor/fragment pair per enumerated lipid molecule.
- Every emitted transition must carry name, mass, adduct, and charge state fields populated.
- Precursor m/z must be reproducible from elemental composition + adduct rule for a hand-checked lipid class (e.g. PC 16:0/18:1 + H+).
- blib export must succeed only after activating the collision energy module and adding transitions, per README instructions.

## Limitations

- Collision energy models are currently exclusive to lipid mediators; other lipid classes require more structurally diverse standards and CE-dependent measurements.
- Current implementation focuses on eukaryotic model systems due to lack of standards for bacteria such as E. coli and archaea.
- No dedicated scoring algorithm for false discovery rates; does not address coeluting/isobaric lipids and peak area integration.
- Retention time, elution order, and isotope labeled standards must be considered separately to avoid mis-annotations.
- No large-scale methods exist to cope with stereoisomers due to the lack of isotope-coded standards.

## Evidence

- [methods] LipidCreator stores per-class rules at the precursor level (lipid category, class name, fatty acyl chain length and type, adducts) so the user defines head groups, adducts, and fatty acyls or long chain bases: [evidence span withheld — anchor status: not_found]
- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [readme] LipidCreator is a plugin for Skyline supporting targeted workflow development in lipidomics. It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline.: [evidence span withheld — anchor status: not_found]
- [readme] Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [abstract] LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries.: "LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries."
