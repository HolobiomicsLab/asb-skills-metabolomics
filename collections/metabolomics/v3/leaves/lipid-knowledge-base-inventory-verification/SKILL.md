---
name: lipid-knowledge-base-inventory-verification
description: Use when a targeted lipidomics assay design must be validated against
  the tool's claimed knowledge-base scope, e.g. LipidCreator transition lists built
  from head-group, fatty-acyl, and adduct rules need per-class inventory tallies compared
  to a reported 61+ classes / 10^12-molecule coverage.
when_to_use_negative:
- Input is already an exported transition list from Skyline/LipidCreator that you
  only need to inspect or edit — no inventory reconstruction needed
- The goal is collision-energy optimization or stable-isotope-labeling management
  rather than verifying knowledge-base enumeration coverage
- The lipidome of interest is bacterial (e.g. E. coli, archaea) — the paper explicitly
  notes lack of standards for acyl-PG, acyl-PE in E.coli and rare lipidome-wide coverage,
  so the eukaryotic-tuned inventory does not apply
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3820
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2640
tools:
- name: LipidCreator
  role: Source of the knowledge-base rules (head groups, fatty acyls, adducts) and
    of the exported transition lists that must be reproduced; used standalone or as
    a Skyline plugin for PRM/MRM target-list creation
  repo: github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Host application into which LipidCreator is integrated; the transition list
    and fragment library are validated inside Skyline small-molecule workflows
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: Panorama
  role: Downstream repository where Skyline-based assay data (validated transition
    lists) can be archived after inventory verification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:1d9eda545fdb4d878b6f0624d7373f3e2289802f8500bdafd55e0dac2d925462
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - Panorama
  - Skyline
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3820
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# lipid-knowledge-base-inventory-verification

## Summary

Verifies that a lipid knowledge base (LipidCreator-style) enumerates the precursor and fragment masses it claims, by reconstructing the head-group + fatty-acyl + adduct assembly logic and checking coverage against the paper's reported scale of 10^12 lipid molecules across >60 lipid classes.

## When to use

Use when a targeted lipidomics assay design must be validated against the tool's claimed knowledge-base scope, e.g. LipidCreator transition lists built from head-group, fatty-acyl, and adduct rules need per-class inventory tallies compared to a reported 61+ classes / 10^12-molecule coverage.

## When NOT to use

- Input is already an exported transition list from Skyline/LipidCreator that you only need to inspect or edit — no inventory reconstruction needed
- The goal is collision-energy optimization or stable-isotope-labeling management rather than verifying knowledge-base enumeration coverage
- The lipidome of interest is bacterial (e.g. E. coli, archaea) — the paper explicitly notes lack of standards for acyl-PG, acyl-PE in E.coli and rare lipidome-wide coverage, so the eukaryotic-tuned inventory does not apply

## Inputs

- LipidCreator knowledge-base tables (lipid classes, molecules, head groups, fatty-acyl chains, adduct rules)
- Per-class fragment definitions (positive and negative polarity)
- Reported coverage figures: >60 lipid classes, 10^12 lipid molecules

## Outputs

- transition_list.csv — precursor m/z, fragment m/z, charge, polarity, lipid class, molecule per enumerated target
- coverage_summary.csv — per-class molecule counts and match flags against the reported >61-class / 10^12-molecule scale

## How to apply

1. Load the lipid-class definitions from the LipidCreator knowledge-base tables — each class carries the lipid category, class name, fatty-acyl chain length and type, and adduct rules. 2. Assemble head groups plus fatty-acyl chains plus adduct per class to compute monoisotopic precursor m/z, mirroring LipidCreator's precursor-level store of per-class rules. 3. Enumerate diagnostic MS/MS fragment ions in both positive and negative polarity using class-specific fragment definitions, so every assembled molecule yields at least one precursor/fragment pair. 4. Emit the transition list (precursor m/z, fragment m/z, charge, polarity, lipid class, molecule) and tally per-class molecule counts from the entity inventory. 5. Compare the tallies to the paper's reported coverage — over 1012 lipid molecules and 61+ lipid classes — writing per-class counts and match flags to a coverage summary. The validation criterion is that the implemented inventory reproduces the reported scale and every enumerated molecule appears in the transition list.

## Related tools

- **LipidCreator** (Source of the knowledge-base rules (head groups, fatty acyls, adducts) and of the exported transition lists that must be reproduced; used standalone or as a Skyline plugin for PRM/MRM target-list creation) — github.com/lifs-tools/lipidcreator
- **Skyline** (Host application into which LipidCreator is integrated; the transition list and fragment library are validated inside Skyline small-molecule workflows) — https://skyline.ms/project/home/software/Skyline/begin.view
- **Panorama** (Downstream repository where Skyline-based assay data (validated transition lists) can be archived after inventory verification)

## Examples

```
mono LipidCreator.exe --export-blib --ce-module on  # then load the exported transition list in Skyline to verify per-class precursor/fragment coverage against the 10^12-molecule inventory
```

## Evaluation signals

- transition_list.csv must contain at least one precursor/fragment pair per enumerated lipid molecule in the inventory
- Per-class tallies in coverage_summary.csv must collectively match the reported scale of >1012 lipid molecules and >60 lipid classes, with per-class match flags agreeing for every class
- Exported blib spectral library (after enabling the collision-energy module) must be openable in Skyline, confirming the fragment definitions are well-formed
- Computed precursor m/z must reproduce monoisotopic values for known reference lipids such as NIST SRM 1950 standard material entries

## Limitations

- The majority of eukaryotic lipid classes are covered 'with some exceptions' (Supplementary Data 1); rare bacterial/archaeal classes lack standards and are not represented
- Collision-energy models currently apply only to lipid mediators; extending them to other classes requires structurally diverse standards and CE-dependent fragmentation measurements
- No large-scale handling of stereoisomers — lacking isotope-coded standards, stereoisomer disambiguation is out of scope
- Fragment masses of lipid mediators were validated against the Metlin database (https://metlin.scripps.edu/); classes without external reference data cannot be validated this way
- Coverage thresholds are from the paper's own reported figures (10^12 molecules, 61+ classes); other tools' knowledge bases may have different scales and this verification logic would need re-anchoring

## Evidence

- [abstract] Coverage reported in abstract: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [results] Per-class rule storage: [evidence span withheld — anchor status: not_found]
- [results] Fragment-mass computation role: [evidence span withheld — anchor status: not_found]
- [readme] Integration with Skyline for target lists: "LipidCreator is a plugin for Skyline supporting targeted workflow development in lipidomics. It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in"
- [discussion] Coverage exceptions caveat: "Here, the majority of lipid classes are covered with some exceptions (Supplementary Data 1)."
