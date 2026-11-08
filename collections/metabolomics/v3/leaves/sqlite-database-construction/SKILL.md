---
name: sqlite-database-construction
description: Use when constructing a MEANtools-compatible SQLite database (retroRules_mass_shifts.sqlite)
  that stores mass-shift-annotated RetroRules rules in loose/medium/strict tiers,
  or when converting a structure database such as LOTUS to SQLite for metabolite assignment.
when_to_use_negative:
- The MEANtools-compatible retroRules_mass_shifts.sqlite or LOTUS SQLite database
  already exists — rebuilding is redundant since construction is a one-time step during
  initial retrieval or updates.
- The task is downstream MEANtools analysis (mutual-rank correlation, ClusterONE feature-cluster
  merging) rather than preparing the underlying rule/structure databases.
- The analysis requires MS/MS spectral library or networking-based annotation, which
  MEANtools' current SQLite-based structure assignment does not support.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MEANtools
  role: Consumer of the constructed SQLite databases; the schema must be compatible
    with MEANtools queries for mass-shift-annotated rules and LOTUS structure lookups
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of the reaction rules and rule-enzyme tiers that are mass-shift-annotated
    and packaged into retroRules_mass_shifts.sqlite
- name: LOTUS
  role: Structure database converted to SQLite format so MEANtools can assign molecular
    structures to mass features via adducts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:a26f115dc8789b7454a568c928c81f3bd56c32d2b2f2adfae2e6f7887c64bc78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sqlite-database-construction

## Summary

Build MEANtools-compatible SQLite databases by attaching mass shifts to RetroRules reaction rules via MetaNetX cross-referencing, packaging the loose/medium/strict rule-enzyme tiers, and converting the LOTUS structure database to SQLite for adduct-based metabolite lookup.

## When to use

Use when constructing a MEANtools-compatible SQLite database (retroRules_mass_shifts.sqlite) that stores mass-shift-annotated RetroRules rules in loose/medium/strict tiers, or when converting a structure database such as LOTUS to SQLite for metabolite assignment.

## When NOT to use

- The MEANtools-compatible retroRules_mass_shifts.sqlite or LOTUS SQLite database already exists — rebuilding is redundant since construction is a one-time step during initial retrieval or updates.
- The task is downstream MEANtools analysis (mutual-rank correlation, ClusterONE feature-cluster merging) rather than preparing the underlying rule/structure databases.
- The analysis requires MS/MS spectral library or networking-based annotation, which MEANtools' current SQLite-based structure assignment does not support.

## Inputs

- RetroRules reaction rules (SQLite-formatted public database)
- MetaNetX identifier mappings for cross-referencing RetroRules reactions
- Rhea and KEGG-orthology database cross-references for the loose tier
- ECDomainMiner prediction entries for the medium tier
- LOTUS structure database release (e.g., 10/10/2023 download) for SQLite conversion

## Outputs

- retroRules_mass_shifts.sqlite (single SQLite database holding mass-shift-annotated rules and the three tiers)
- Combined_small_middle_big_datasets.csv (loose, medium, and strict datasets in one file)
- LOTUS SQLite database compatible with MEANtools
- adduct_structure_lookup.csv (adduct-to-structure lookup table produced downstream)

## How to apply

Retrieve the RetroRules reaction rules (publicly available as an SQLite database usable directly with MEANtools) and cross-reference each reaction against MetaNetX identifier mappings to attach a mass shift; this is a one-time step performed during initial retrieval or updates. Assemble the three reaction-rule-enzyme association tiers: the loose dataset (all RetroRules rules-enzyme associations cross-referenced with Rhea and KEGG-orthology, 2,704,948 entries), the medium dataset (experimentally validated entries plus ECDomainMiner predictions, 429,267 entries), and the strict dataset (experimentally validated entries only, 67,501 entries). Store the mass-shift-annotated rules and the three tiers in a single SQLite file (retroRules_mass_shifts.sqlite); the tiered datasets are also distributable as a single CSV (Combined_small_middle_big_datasets.csv on Zenodo). Separately, convert the chosen LOTUS release (e.g., downloaded 10/10/2023) to SQLite and publish it in the tool's GitHub repository so MEANtools can assign structures to mass features by identifying roughly 40 default adducts and querying candidate neutral masses within the default 20 ppm tolerance.

## Related tools

- **MEANtools** (Consumer of the constructed SQLite databases; the schema must be compatible with MEANtools queries for mass-shift-annotated rules and LOTUS structure lookups) — github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of the reaction rules and rule-enzyme tiers that are mass-shift-annotated and packaged into retroRules_mass_shifts.sqlite)
- **LOTUS** (Structure database converted to SQLite format so MEANtools can assign molecular structures to mass features via adducts)

## Evaluation signals

- Verify the tier entry counts match expectations: loose 2,704,948, medium 429,267, strict 67,501.
- Confirm retroRules_mass_shifts.sqlite opens and returns mass-shift-annotated rules for all three tiers.
- Confirm adduct-based lookups return non-empty structure assignments for mass features matched within the 20 ppm tolerance.
- Check that every rule carries a mass shift after MetaNetX cross-referencing (no unmapped reactions remain).
- Confirm the published tiered CSV on Zenodo contains all three datasets in a single file.

## Limitations

- RetroRules-based rules are designed for single-substrate reactions only, limiting the reactions representable in the database.
- The loose tier retains spurious reaction-Pfam links (e.g., AminoTran_1_2, Glyco_transf_20) after the Pfam cutoff of 6, so the strict dataset is needed for confident predictions.
- Stereomeric-conversion rules exist but MEANtools filters them, and mass spectrometric data cannot capture stereoisomers.
- The LOTUS SQLite build is tied to a dated snapshot (10/10/2023); releases must be explicitly versioned.
- Accuracy of downstream adduct-based matching depends on mass spectrometer resolution, which bounds the usefulness of the 20 ppm default tolerance.

## Evidence

- [discussion] RetroRules SQLite availability: "The RetroRules database is publicly available as SQLite database and can be used directly with MEANtools."
- [discussion] LOTUS SQLite conversion: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
- [discussion] Tiered datasets single CSV on Zenodo: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
- [supplementary] Loose dataset definition: "Loose dataset contains reaction rules-enzyme associations from the RetroRules database"
- [discussion] 20 ppm default tolerance: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [readme] README documentation pointer: "MEANtools is licensed under the [MIT License](LICENSE.md)"
