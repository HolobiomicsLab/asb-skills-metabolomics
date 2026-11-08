---
name: compound-identity-collision-handling-smiles-vs-formula
description: Use when a metabolite annotation search returns multiple compounds at
  one m/z — e.g. identically-named or same-formula hits — and you must decide whether
  rows represent distinct structures or merely the same molecular formula before ranking
  or exporting annotation tables.
when_to_use_negative:
- Raw peak data — MetaboShiny does not accept unprocessed spectra; preprocessing with
  XCMS or MSnbase must happen first.
- Only one compound database is needed and the user does not care about distinguishing
  isomeric structures — a simpler m/z lookup without SMILES collapse suffices.
- Data is not mass spectrometry metabolomics, or no compound database with SMILES
  fields is available.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetaboShiny
  role: Interactive R/RShiny app that performs the m/z-to-compound database search,
    applies SMILES-based collapse, isotope/adduct prioritization and ppm-error sorting,
    and exports the annotation table.
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package that builds the compound databases (over 30 supported) storing
    m/z values for common isotopes and adducts of every compound, plus name/formula/SMILES/ID
    fields that the collision handler queries.
  repo: https://github.com/joannawolthuis/MetaboShiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:c36131568d1e74db14f3a540090f3bc87c0093c6e743c670e05d9eddf51f5390
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 38
---

# compound identity collision handling — SMILES vs formula

## Summary

When database search returns multiple m/z matches that collapse into one row, MetaboShiny collapses only compounds sharing an identical SMILES into a single row, while compounds with the same molecular formula but different structures (different SMILES) are kept as separate rows. This means each retained row represents a unique chemical structure rather than a unique chemical formula, and isotope/adduct status is used to further prioritize between colliding identities.

## When to use

Use when a metabolite annotation search returns multiple compounds at one m/z — e.g. identically-named or same-formula hits — and you must decide whether rows represent distinct structures or merely the same molecular formula before ranking or exporting annotation tables.

## When NOT to use

- Raw peak data — MetaboShiny does not accept unprocessed spectra; preprocessing with XCMS or MSnbase must happen first.
- Only one compound database is needed and the user does not care about distinguishing isomeric structures — a simpler m/z lookup without SMILES collapse suffices.
- Data is not mass spectrometry metabolomics, or no compound database with SMILES fields is available.

## Inputs

- Query m/z list (positive/negative peaklist, e.g. CSV of m/z and per-sample intensities)
- SQLite-backed compound database with fields: name, molecular formula, SMILES, database ID, adduct-derived m/z, isotope and adduct labels
- User-configured mass-spec error margin in ppm

## Outputs

- Annotation table (mz_annotation_matches.csv) with one row per unique chemical structure (identical SMILES collapsed) and separate rows for same-formula/different-SMILES compounds, retaining name, molecular formula, SMILES, database ID, adduct and dppm columns
- Per-compound priority/score reflecting isotope and adduct status

## How to apply

Run the database search over the SQLite-backed compound db (built by MetaDBparse) with a ppm window set by the user-defined mass-spec error margin. When two or more matched compounds resolve to the same structure, collapse them into one row, keeping the adduct and isotope-percentage fields; when they share a formula but differ in SMILES, keep them as separate rows so downstream priority ranking can distinguish them. After collapsing, filter to main-peak isotopes first, then sort by ppm error so the best-supported structure per m/z rises to the top. MetaboShiny additionally prioritizes compounds by their isotope and adduct status (e.g. M + H vs. M + H-H2O) so a single putative identity is reported per m/z where possible.

## Related tools

- **MetaboShiny** (Interactive R/RShiny app that performs the m/z-to-compound database search, applies SMILES-based collapse, isotope/adduct prioritization and ppm-error sorting, and exports the annotation table.) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package that builds the compound databases (over 30 supported) storing m/z values for common isotopes and adducts of every compound, plus name/formula/SMILES/ID fields that the collision handler queries.) — https://github.com/joannawolthuis/MetaboShiny

## Examples

```
library(MetaboShiny); start_metshi(inBrowser=T)
```

## Evaluation signals

- Searching 264 m/z against HMDB with an appropriate ppm margin returns creatine riboside as a putative hit with adduct M + H, demonstrating the search+collapse pipeline resolves a known identity.
- Exported annotation table (mz_annotation_matches.csv) contains exactly one row per unique SMILES; rows with identical molecular formula but distinct SMILES remain un-collapsed.
- 'Adduct' and 'dppm' columns are present in the output and non-null for the retained row.
- After filtering to main-peak isotopes and sorting by ppm error, the top row for a given m/z is the structure with the lowest absolute ppm error and a compatible adduct (e.g. M + H in positive mode).
- Compounds that are structurally identical but listed under multiple database IDs occupy one row, not several.

## Limitations

- Structures with identical SMILES but differing source-database metadata are merged, so source-database provenance can be lost for colliding identities.
- Distinguishing isomers relies on SMILES strings being populated correctly in the underlying database; missing or non-canonical SMILES will defeat the collapse step.
- Putative annotation (MSI level 2) — identity assignments are not structure-confirmed, and ppm-error sorting can only rank, not confirm, the correct isomer.
- Users cannot automatically update databases; an outdated db may miss recently added isomer entries.

## Evidence

- [methods] Collapse hits sharing identical SMILES into single rows while keeping compounds with identical formulas but different structures/SMILES as separate rows.: [evidence span withheld — anchor status: not_found]
- [methods] Filter the matches to main-peak isotopes first, then sort by ppm error, retaining adduct and isotope-percentage fields.: [evidence span withheld — anchor status: not_found]
- [supplementary] MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status: "MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status"
- [readme] The MetaDBparse package containing all the search and compound-db specific functionality without the Shiny user interface is available from the joannawolthuis/MetaDBparse repository: "The MetaDBparse package containing all the search and compound-db specific functionality without the Shiny user interface is available from the joannawolthuis/MetaDBparse repository"
- [methods] a search including the HMDB uncovered creatine riboside (M + H adduct) as a putative hit: [evidence span withheld — anchor status: not_found]
