---
name: reaction-rule-mass-shift-annotation
description: 'Use when integrating unlabeled metabolome mass features with reaction-rule-based
  pathway prediction: you need neutral-mass candidates per m/z feature and mass shifts
  attached to RetroRules reaction rules before filtering by correlated enzymes.'
when_to_use_negative:
- Annotating mass features from MS/MS spectra — MEANtools currently supports no MS/MS
  data loading or spectral library/networking-based annotation; use a spectral library
  tool instead.
- Reactions involving stereoisomers or multiple substrates — RetroRules-based rules
  are designed for single-substrate reactions only, and MEANtools filters stereomeric
  conversion rules to avoid complexity.
- Generating all product molecules for every RetroRules reaction exhaustively — with
  the substantial number of cataloged rules this is time-consuming and computationally
  intensive; the adduct/mass-shift lookup strategy exists precisely to avoid it.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MEANtools
  role: Pipeline that consumes the mass-shift-annotated RetroRules SQLite database
    and performs adduct-based structure assignment of mass features via LOTUS queries
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of general reaction rules and rules-enzyme association tiers; publicly
    available as SQLite and used directly with MEANtools
- name: LOTUS
  role: Structure database (10/10/2023 download, converted to SQLite) queried by candidate
    neutral mass to assign structures to mass features
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: MetaNetX
  role: Identifier mappings used to cross-reference RetroRules reactions and attach
    a mass shift to every rule (one-time step)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:94ef6d3e0ddb21ee7a00bace9a63bb8a3448a32f2cb7b5a848eaf92ca63e82ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - LOTUS
  - MEANtools
  - MetaNetX
  - RetroRules
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# reaction-rule-mass-shift-annotation

## Summary

Builds a mass-shift-annotated reaction-rule database by cross-referencing RetroRules reactions against MetaNetX identifier mappings, then assigns putative molecular structures to metabolome mass features by enumerating adducts and querying the LOTUS SQLite database within a 20 ppm mass accuracy tolerance.

## When to use

Use when integrating unlabeled metabolome mass features with reaction-rule-based pathway prediction: you need neutral-mass candidates per m/z feature and mass shifts attached to RetroRules reaction rules before filtering by correlated enzymes.

## When NOT to use

- Annotating mass features from MS/MS spectra — MEANtools currently supports no MS/MS data loading or spectral library/networking-based annotation; use a spectral library tool instead.
- Reactions involving stereoisomers or multiple substrates — RetroRules-based rules are designed for single-substrate reactions only, and MEANtools filters stereomeric conversion rules to avoid complexity.
- Generating all product molecules for every RetroRules reaction exhaustively — with the substantial number of cataloged rules this is time-consuming and computationally intensive; the adduct/mass-shift lookup strategy exists precisely to avoid it.

## Inputs

- RetroRules reaction rules (SQLite database, usable directly with MEANtools)
- MetaNetX identifier mappings for cross-referencing reaction mass shifts
- Rhea and KEGG-orthology cross-references for the loose rules-enzyme tier
- ECDomainMiner predictions for the medium rules-enzyme tier
- Metabolome mass feature m/z table (feature list from FTICR-MS or Orbitrap data)
- LOTUS structure database (10/10/2023 release, converted to SQLite)

## Outputs

- retroRules_mass_shifts.sqlite — single SQLite database holding mass-shift-annotated rules and the three tiered rules-enzyme datasets (loose/medium/strict)
- adduct_structure_lookup.csv — adduct-to-structure lookup assigning molecular structures to each mass feature

## How to apply

Retrieve the RetroRules reaction rules (publicly available as an SQLite database usable directly with MEANtools) and cross-reference its reactions against MetaNetX identifier mappings to attach a mass shift to every rule; assemble the three rules-enzyme association tiers (loose: all RetroRules rules cross-referenced with Rhea and KEGG-orthology, ~2,704,948 entries; medium: experimentally validated plus ECDomainMiner predictions, ~429,267 entries; strict: experimentally validated only, ~67,501 entries) and store them with the mass-shift-annotated rules in a single SQLite file (retroRules_mass_shifts.sqlite). For each metabolome mass feature, enumerate the default adduct list of roughly 40 adducts and compute candidate neutral masses within the default 20 ppm tolerance, chosen to balance sensitivity and specificity. Query the SQLite conversion of the LOTUS structure database (10/10/2023 release) with the candidate neutral masses to assign structures to each mass feature, writing the adduct-to-structure lookup table (adduct_structure_lookup.csv).

## Related tools

- **MEANtools** (Pipeline that consumes the mass-shift-annotated RetroRules SQLite database and performs adduct-based structure assignment of mass features via LOTUS queries) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of general reaction rules and rules-enzyme association tiers; publicly available as SQLite and used directly with MEANtools)
- **LOTUS** (Structure database (10/10/2023 download, converted to SQLite) queried by candidate neutral mass to assign structures to mass features) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **MetaNetX** (Identifier mappings used to cross-reference RetroRules reactions and attach a mass shift to every rule (one-time step))

## Evaluation signals

- retroRules_mass_shifts.sqlite opens and returns mass-shift-annotated rules for all three tiers with the expected entry counts (loose ≈ 2,704,948; medium ≈ 429,267; strict ≈ 67,501).
- Adduct-based lookups return non-empty structure assignments for mass features matched within 20 ppm.
- Every RetroRules rule carries an attached mass shift after the MetaNetX cross-reference step.
- adduct_structure_lookup.csv is produced and covers each mass feature in the input metabolome feature table.
- Tier dataset sizes match the Combined_small_middle_big_datasets.csv released at zenodo.org/records/15697913.

## Limitations

- The 20 ppm default tolerance ties annotation confidence to mass spectrometer resolution: the resolution directly impacts the precision of detected m/z values and the confidence of adduct-based matching.
- Loose RetroRules ruleset retains spurious reaction-Pfam links after the Pfam cutoff of 6, so the strict dataset should be preferred for pathway prediction.
- Mass spectrometric data does not capture stereoisomers, which hinders reconstruction of steps such as phenylalanine to p-coumaroyl-CoA.
- RetroRules-based rules cover single-substrate reactions only, so multi-substrate conversions (e.g., tyrosine + p-coumaroyl-CoA to p-coumaroyltyramine) cannot be modeled.
- Annotation could be improved by allowing MS/MS data loading and mass spectral library/networking-based annotation approaches to increase accuracy.

## Evidence

- [discussion] It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023).: "It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023)."
- [discussion] The RetroRules database is publicly available as SQLite database and can be used directly with MEANtools.: "The RetroRules database is publicly available as SQLite database and can be used directly with MEANtools."
- [discussion] The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository.: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
- [discussion] By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity.: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [discussion] The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913.: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
- [supplementary] The strict dataset contains only experimentally validated entries: "The strict dataset contains only experimentally validated entries"
