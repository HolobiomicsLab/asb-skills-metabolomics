---
name: isotope-adduct-duplicate-removal
description: Use when an xcms/CAMERA feature table contains multiple m/z features
  per compound (adducts, isotopes) and a compound-level count is needed before applying
  fold-change, intensity, or sample-group filters.
when_to_use_negative:
- Input is a raw spectral file (mzML) that has not yet been feature-detected and grouped
  by XCMS/CAMERA — run feature detection first.
- The goal is isotope-tracing analysis of labeled compounds — Metaboseek's Label Finder,
  not adduct/isotope removal, is the appropriate module.
- Features have no CAMERA group annotations, since deduplication is defined over CAMERA-annotated
  feature groups.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Metaboseek
  role: Provides the feature-table processing GUI in which CAMERA-annotated adduct/isotope
    duplicates are removed from the loaded WT vs hacl-1 feature table
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Used inside Metaboseek for feature detection and feature grouping; its output
    feature table with CAMERA annotations is the input to deduplication
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used with xcms to load xcms results into Metaboseek as a Feature Table prior
    to deduplication and filtering
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: MassTools
  role: R package for mass and molecular formula calculations used alongside Metaboseek
    feature-table processing
  repo: https://github.com/mjhelf/MassTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:cdeb6643e0646524edebe1132e7da6391e9ddafdc3acb7f14d525fa5a11899c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - MassTools
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# isotope-adduct-duplicate-removal

## Summary

A Metaboseek-based deduplication step that removes adduct and isotope duplicates arising from CAMERA-annotated feature groups in an LC/MS feature table, so that each detected compound (e.g., the 32 CA-derived metabolites in the hacl-1 study) is represented by a single feature before downstream filtering or annotation.

## When to use

Use when an xcms/CAMERA feature table contains multiple m/z features per compound (adducts, isotopes) and a compound-level count is needed before applying fold-change, intensity, or sample-group filters.

## When NOT to use

- Input is a raw spectral file (mzML) that has not yet been feature-detected and grouped by XCMS/CAMERA — run feature detection first.
- The goal is isotope-tracing analysis of labeled compounds — Metaboseek's Label Finder, not adduct/isotope removal, is the appropriate module.
- Features have no CAMERA group annotations, since deduplication is defined over CAMERA-annotated feature groups.

## Inputs

- xcms/CAMERA feature table (.mskFT preferred, or .csv with 'mz' and 'rt' columns) with CAMERA group annotations
- Raw LC/MS data already processed by the Metaboseek XCMS module with CAMERA feature grouping

## Outputs

- Deduplicated feature table (.mskFT with embedded processing history) with one feature per compound group
- Compound-level feature counts usable for reporting (e.g., 32 CA-derived metabolites)

## How to apply

Load the feature table into Metaboseek with LoadTableModule (files with 'mz' and 'rt' columns are supported as .csv, with .mskFT preferred for embedded metadata). The feature table should already carry CAMERA group annotations produced by the integrated XCMS module during feature detection and grouping. In the feature-table processing chain, remove adduct and isotope duplicates within each CAMERA-annotated feature group so each compound is represented once — in the hacl-1 vs WT CA-supplementation study this reduced duplicated features to the reported set of 32 CA-dependent metabolites. Deduplicate before applying the MultiFilterModule (fold change, intensity, group filters), since duplicates would inflate the compound count and skew comparative statistics. Save the deduplicated table with SaveTableModule in .mskFT format to preserve the complete processing history, and validate compound-level features by inspecting a known compound (e.g., the m/z 185.1183 alpha-hydroxylated citronellic acid derivative showing a formic-acid neutral loss) in the MS2 Browser.

## Related tools

- **Metaboseek** (Provides the feature-table processing GUI in which CAMERA-annotated adduct/isotope duplicates are removed from the loaded WT vs hacl-1 feature table) — https://github.com/mjhelf/Metaboseek
- **xcms** (Used inside Metaboseek for feature detection and feature grouping; its output feature table with CAMERA annotations is the input to deduplication) — https://github.com/sneumann/xcms
- **MSnbase** (Used with xcms to load xcms results into Metaboseek as a Feature Table prior to deduplication and filtering) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **MassTools** (R package for mass and molecular formula calculations used alongside Metaboseek feature-table processing) — https://github.com/mjhelf/MassTools

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- After deduplication, the filtered WT vs hacl-1 CA-supplemented table contains the reported 32 CA-derived metabolites (Supplementary Fig. 3), not a larger duplicate-inflated count.
- Each CAMERA-annotated feature group contributes at most one representative feature in the saved table.
- The saved .mskFT file preserves the complete processing history including the deduplication step.
- A representative compound-level feature (m/z 185.1183) shows the characteristic neutral loss of formic acid in its MS/MS spectrum in the MS2 Browser.
- Fold-change, intensity, and group filters applied after deduplication yield counts consistent with the published compound-level result rather than feature-level counts.

## Limitations

- Deduplication depends on CAMERA annotation quality; mis-grouped features will leave duplicates or remove true compounds.
- Metabolite annotations after this workflow are limited to confidence Level 3 (tentative structure or compound class assignments only).
- Many detected features remain unidentified after deduplication; the space of yet unidentified biogenic small molecules remains large.

## Evidence

- [methods] Remove adduct and isotope duplicates arising from CAMERA-annotated feature groups so each CA-derived compound is represented once.: [evidence span withheld — anchor status: not_found]
- [intro] Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation): "Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation)"
- [readme] LoadTableModule supports .csv with 'mz' and 'rt' columns: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
- [results] a combination of fold change, intensity, and sample group-dependent filters revealed 32 CA-dependent metabolites: [evidence span withheld — anchor status: not_found]
