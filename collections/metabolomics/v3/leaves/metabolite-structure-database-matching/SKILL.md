---
name: metabolite-structure-database-matching
description: Use when a metabolomics mass feature list must be annotated with candidate
  structures by enumerating adducts (default ~40) and matching neutral masses against
  a structure database within a 20 ppm tolerance.
when_to_use_negative:
- 'When MS/MS spectra or a spectral library are available: MEANtools does not support
  loading MS/MS data or mass spectral library/networking-based annotation, which would
  give higher-confidence identifications than exact-mass matching alone'
- 'When stereoisomers must be distinguished: mass spectrometric data does not capture
  stereoisomers, so this matching cannot resolve them (e.g., phenylalanine vs p-coumaroyl-CoA
  stereoisomer cases in the article)'
- When working at low mass-spectrometer resolution, since the precision of detected
  m/z values directly limits the confidence of adduct-based matching within the 20
  ppm window
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MEANtools
  role: Pipeline step that identifies possible adducts per mass feature, computes
    candidate neutral masses at 20 ppm tolerance, and queries the structure database;
    writes adduct_structure_lookup.csv
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: LOTUS
  role: Structure database queried by molecular weight; the SQLite conversion compatible
    with MEANtools (downloaded 10/10/2023) is distributed in the MEANtools GitHub
    repository
  repo: github.com/IntegrativeBioinformaticsLab/meantools
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:23487629f33e07b7c23c2626e1ca37ba90b4bbbff175ef94f0d6f6ea76a9e67b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# metabolite-structure-database-matching

## Summary

Assigns candidate molecular structures to metabolomic mass features by enumerating adducts, back-calculating candidate neutral masses, and querying the LOTUS structure database (SQLite conversion, 10/10/2023 release) by molecular weight within a 20 ppm mass accuracy tolerance — the structure-annotation stage of the MEANtools pipeline.

## When to use

Use when a metabolomics mass feature list must be annotated with candidate structures by enumerating adducts (default ~40) and matching neutral masses against a structure database within a 20 ppm tolerance.

## When NOT to use

- When MS/MS spectra or a spectral library are available: MEANtools does not support loading MS/MS data or mass spectral library/networking-based annotation, which would give higher-confidence identifications than exact-mass matching alone
- When stereoisomers must be distinguished: mass spectrometric data does not capture stereoisomers, so this matching cannot resolve them (e.g., phenylalanine vs p-coumaroyl-CoA stereoisomer cases in the article)
- When working at low mass-spectrometer resolution, since the precision of detected m/z values directly limits the confidence of adduct-based matching within the 20 ppm window

## Inputs

- List of metabolome mass features (m/z values) from a metabolomics dataset
- LOTUS structure database in SQLite format (downloaded 10/10/2023, distributed with MEANtools), or a user-defined metabolite structure database
- Adduct list (default set of roughly 40 adducts)

## Outputs

- Adduct-to-structure lookup table (adduct_structure_lookup.csv) mapping each mass feature to candidate structures with their adduct and neutral mass
- Per-mass-feature structure assignments (matched vs unmatched candidates) usable in downstream MEANtools pathway reconstruction

## How to apply

For each metabolome mass feature, enumerate the default adduct list of roughly 40 adducts and compute candidate neutral masses within the default 20 ppm mass accuracy tolerance, which the authors chose as a balance between sensitivity and specificity. Query the LOTUS database in the SQLite format distributed with the MEANtools GitHub repository (version downloaded 10/10/2023) with these candidate neutral masses to retrieve matching structures. Alternatively, substitute a user-defined metabolite structure database when LOTUS is inappropriate for the study system. Write the resulting adduct-to-structure lookup table (adduct_structure_lookup.csv) so downstream MEANtools steps can link predicted structures to mass features. Interpretation must respect that this is an exact-mass match: stereoisomers are not distinguished, and confidence scales with the resolution of the mass spectrometer that produced the m/z values.

## Related tools

- **MEANtools** (Pipeline step that identifies possible adducts per mass feature, computes candidate neutral masses at 20 ppm tolerance, and queries the structure database; writes adduct_structure_lookup.csv) — github.com/IntegrativeBioinformaticsLab/meantools
- **LOTUS** (Structure database queried by molecular weight; the SQLite conversion compatible with MEANtools (downloaded 10/10/2023) is distributed in the MEANtools GitHub repository) — github.com/IntegrativeBioinformaticsLab/meantools

## Evaluation signals

- Non-empty structure assignments must be returned for mass features matched within 20 ppm of a LOTUS entry
- Every assigned candidate neutral mass must fall within ±20 ppm of the observed m/z after accounting for the assigned adduct
- The adduct_structure_lookup.csv must cover the default adduct set (~40 adducts) for each mass feature processed
- Benchmark reproducibility: loading the 374 characterized substrate/product structures (Combined_small_middle_big_datasets.csv, Zenodo 15697913) against the same LOTUS SQLite release should recover 132 matched structures, chi-square 32.353 (DF=1, p < 0.001)
- Substituting a user-defined database in place of LOTUS should still produce the same lookup-table schema and per-feature assignment format

## Limitations

- Mass spectrometric data does not capture stereoisomers, so structurally distinct stereoisomer candidates cannot be separated by this mass-based matching
- No MS/MS data loading or spectral library/networking-based annotation is currently supported, limiting annotation accuracy relative to spectrum-based approaches
- The precision of detected m/z values depends on mass spectrometer resolution, which directly impacts the confidence of adduct-based matching
- Matches are candidate assignments only; downstream filtering (e.g., cross-checking reaction-enzyme associations) is required before pathway-level conclusions

## Evidence

- [discussion] It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023).: "It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023)."
- [discussion] By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity.: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [discussion] The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository.: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
- [discussion] Annotating mass signatures with predicted structures can be improved by allowing to load MS/MS data and use mass spectral library and networking-based annotation approaches [52] to increase accuracy: "Annotating mass signatures with predicted structures can be improved by allowing to load MS/MS data and use mass spectral library and networking-based annotation approaches [52] to increase accuracy"
