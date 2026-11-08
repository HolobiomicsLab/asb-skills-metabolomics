---
name: cross-database-identifier-mapping
description: Use when assembling a unified reaction-rule/metabolite-structure resource
  from separately maintained databases (e.g., RetroRules rules lacking mass shifts,
  LOTUS entries not in SQLite) before rule- or mass-based metabolite annotation.
when_to_use_negative:
- Mass features already have assigned molecular structures — annotation via adduct/LOTUS
  lookups would be redundant.
- The goal is MS/MS spectral-library or molecular-network-based annotation, which
  MEANtools does not currently support.
- Analysis targets multi-substrate reactions or stereoisomeric conversions — RetroRules-based
  rules are single-substrate and MEANtools filters stereomeric rules.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MEANtools
  role: Consumer of the mapped databases; requires RetroRules as SQLite and LOTUS
    converted to SQLite for rule-based and mass-based annotation
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of general reaction rules, cross-referenced via MetaNetX, Rhea, and
    KEGG orthology
- name: LOTUS
  role: Source of metabolite structures, converted to SQLite for molecular-weight
    queries during adduct-based structure assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:37fc8fe85c397173142e762f2eb05bf857d9eafc1bbb244906db0b6ae4d23160
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-database-identifier-mapping

## Summary

Cross-references reaction and metabolite identifiers between public databases (RetroRules↔MetaNetX↔Rhea/KEGG orthology, and LOTUS converted to SQLite) so that reaction rules, mass shifts, enzyme associations, and metabolite structures become interoperable within MEANtools.

## When to use

Use when assembling a unified reaction-rule/metabolite-structure resource from separately maintained databases (e.g., RetroRules rules lacking mass shifts, LOTUS entries not in SQLite) before rule- or mass-based metabolite annotation.

## When NOT to use

- Mass features already have assigned molecular structures — annotation via adduct/LOTUS lookups would be redundant.
- The goal is MS/MS spectral-library or molecular-network-based annotation, which MEANtools does not currently support.
- Analysis targets multi-substrate reactions or stereoisomeric conversions — RetroRules-based rules are single-substrate and MEANtools filters stereomeric rules.

## Inputs

- RetroRules reaction rules (SQLite database)
- MetaNetX identifier mappings
- Rhea database cross-references
- KEGG orthology database cross-references
- ECDomainMiner predictions (for the medium tier)
- LOTUS structure database (downloaded 10/10/2023)

## Outputs

- retroRules_mass_shifts.sqlite (mass-shift-annotated rules plus loose/medium/strict tiered rule-enzyme datasets)
- Combined_small_middle_big_datasets.csv (three-tier ruleset file on Zenodo)
- SQLite-converted LOTUS structure database
- adduct-to-structure lookup table (adduct_structure_lookup.csv)

## How to apply

Retrieve the RetroRules reaction rules (publicly available as a SQLite database usable directly with MEANtools) and cross-reference its reactions against MetaNetX identifier mappings to attach a mass shift to every rule; build this substructure map once, during initial retrieval or updates. Assemble the reaction-rule-to-enzyme association tiers by cross-referencing RetroRules associations with Rhea and KEGG-orthology identifiers, producing the loose (2,704,948 entries), medium (429,267, experimentally validated plus ECDomainMiner predictions), and strict (67,501, experimentally validated only) datasets, and store them in a single SQLite database (retroRules_mass_shifts.sqlite). Convert the metabolite structure database (LOTUS, downloaded 10/10/2023) to SQLite format for compatibility with MEANtools, so mass features can be matched by enumerating ~40 adducts within a 20 ppm tolerance and querying by molecular weight.

## Related tools

- **MEANtools** (Consumer of the mapped databases; requires RetroRules as SQLite and LOTUS converted to SQLite for rule-based and mass-based annotation) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of general reaction rules, cross-referenced via MetaNetX, Rhea, and KEGG orthology)
- **LOTUS** (Source of metabolite structures, converted to SQLite for molecular-weight queries during adduct-based structure assignment)

## Evaluation signals

- Verify tier sizes of the mapped rule-enzyme associations: 2,704,948 (loose), 429,267 (medium), and 67,501 (strict) entries.
- Confirm retroRules_mass_shifts.sqlite opens and returns mass-shift-annotated rules for all three tiers with the expected entry counts.
- Confirm adduct-based lookups return non-empty structure assignments for mass features matched within the 20 ppm tolerance.
- Check that every RetroRules reaction has a mass shift attached after the MetaNetX cross-referencing step.
- Validate the SQLite-converted LOTUS database returns structure records for molecular-weight queries derived from ~40 enumerated adducts.

## Limitations

- The loose ruleset retains spurious reaction–Pfam links even after applying a Pfam cutoff of 6, so the strict dataset should be preferred when specificity matters.
- RetroRules-based rules support single-substrate reactions only, and stereoisomers are not captured by mass spectrometric data, limiting pathway reconstruction (e.g., phenylalanine to p-coumaroyl-CoA).
- Mass spectrometer resolution directly impacts the precision of detected m/z values and the confidence of adduct-based matching at the 20 ppm tolerance.
- Changing ClusterONE parameters or the mapping upstream alters downstream clustering of mass features and transcripts.

## Evidence

- [discussion] The RetroRules database is publicly available as SQLite database and can be used directly with MEANtools.: "The RetroRules database is publicly available as SQLite database and can be used directly with MEANtools."
- [discussion] It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023).: "It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023)."
- [supplementary] reaction rules-enzyme associations from the RetroRules database, cross-referenced with the Rhea and KEGG-orthology database: "reaction rules-enzyme associations from the RetroRules database, cross-referenced with the Rhea and KEGG-orthology database"
- [discussion] By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity.: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [discussion] The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository.: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
