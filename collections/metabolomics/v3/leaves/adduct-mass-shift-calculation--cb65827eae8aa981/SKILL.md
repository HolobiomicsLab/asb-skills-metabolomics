---
name: adduct-mass-shift-calculation
description: Use when preparing an m/z annotation reference index from SDF-derived
  compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS) where each neutral mass must be
  expanded into ion masses for the polarity and adduct set being searched.
when_to_use_negative:
- Data are high-resolution spectra already annotated by METASPACE/pySM, which the
  article notes are optimised for high-resolution instruments (FT-ICR, orbitrap) rather
  than TOF data
- You need MS/MS spectral matching rather than MS1 exact-mass adduct matching — the
  complementary pseudo-MS/MS pathway uses the ms1_id/GNPS spectral library (gnps_minmz100.pkl)
  instead
- Input peaks already carry metabolite assignments and only downstream refinement
  (RefineLipids, CalculateAnnotationStatistics) is required
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SpaMTP
  role: R package whose annotation engine (AdductRules, MALDIMatrixRules, MALDIMatrixProfiles,
    BuildMZAnnotationIndex) performs the adduct mass-shift expansion and filtering
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: R
  role: Runtime environment in which the SpaMTP adduct-rule functions are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:3b482885fa3b743dd95daa122e06c0946cd3df2a793d505347e6ddab31434891
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - SpaMTP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Adduct mass-shift calculation

## Summary

Enumerates common positive- and negative-mode adducts for each neutral-mass entry in a compound reference table so that detected m/z values can be matched by exact mass. SpaMTP pre-computes 47 adduct rules (plus MALDI matrix–adduct rules such as FMP-10) and appends the resulting ion masses to the annotation index used by BuildMZAnnotationIndex/AnnotateBigData.

## When to use

Use when preparing an m/z annotation reference index from SDF-derived compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS) where each neutral mass must be expanded into ion masses for the polarity and adduct set being searched.

## When NOT to use

- Data are high-resolution spectra already annotated by METASPACE/pySM, which the article notes are optimised for high-resolution instruments (FT-ICR, orbitrap) rather than TOF data
- You need MS/MS spectral matching rather than MS1 exact-mass adduct matching — the complementary pseudo-MS/MS pathway uses the ms1_id/GNPS spectral library (gnps_minmz100.pkl) instead
- Input peaks already carry metabolite assignments and only downstream refinement (RefineLipids, CalculateAnnotationStatistics) is required

## Inputs

- Uniform compound reference table derived from SDF databases (ChEBI, HMDB, GNPS, LIPIDMAPS) with neutral masses, formulas, and identifiers
- Polarity setting (positive/negative) and user-defined adduct set
- Optional MALDI matrix specification (e.g. FMP-10) for matrix-adduct rules

## Outputs

- Expanded annotation index with 47 adduct ion masses per neutral-mass entry
- Adduct rule table (ion masses, molecular multiplier, charge, proton-loss count, chemical class, prior) from AdductRules()
- Element-filtered reference table ready for exact-mass m/z matching

## How to apply

First convert the compound SDF databases into a uniform reference table, dropping N/A and zero-value entries. Then calculate 47 common adducts in both positive and negative mode for every neutral-mass entry and append the resulting ion masses to the reference table, so that later exact-mass matching at a user-defined ppm error (e.g. ±3 ppm benchmarked on the human lung data) can match observed peaks to any plausible ion form. Inspect which rules the engine will apply using AdductRules(), which returns ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior; when using a MALDI matrix, generate matrix-specific rules with MALDIMatrixRules (e.g. matrix 'FMP-10', polarity 'positive') and check the audited registry via MALDIMatrixProfiles(). Finally filter the expanded index by user-defined polarity, adduct set, and the elemental-composition constraint limited to common natural-product elements (C, H, N, S, O, P, Br, F, Na, I, Si) before running BuildMZAnnotationIndex/QueryMZAnnotationIndex. Rationale: annotation is platform-agnostic exact-mass matching, so correct adduct expansion directly determines which m/z values can receive level 4 (isomer-listed) annotations.

## Related tools

- **SpaMTP** (R package whose annotation engine (AdductRules, MALDIMatrixRules, MALDIMatrixProfiles, BuildMZAnnotationIndex) performs the adduct mass-shift expansion and filtering) — github.com/GenomicsMachineLearning/SpaMTP
- **R** (Runtime environment in which the SpaMTP adduct-rule functions are executed)

## Examples

```
fmp10_rules <- MALDIMatrixRules(maldi_matrix = "FMP-10", polarity = "positive")
```

## Evaluation signals

- Number of appended ion-form rows equals number of neutral entries × selected adducts (47 common adducts across both polarities when fully expanded)
- Every adduct row carries a valid ion mass, molecular multiplier, charge, proton-loss count, and chemical class as returned by AdductRules()
- Elemental composition of all retained entries is limited to C, H, N, S, O, P, Br, F, Na, I, Si after filtering
- MALDI matrix rules match the audited registry entries from MALDIMatrixProfiles() for the chosen matrix (e.g. FMP-10) and polarity
- Downstream spot check: m/z values from the mouse liver spotted-standards dataset annotate at the benchmark ±3 ppm and reproduce the ~90% name agreement with METASPACE (HMDB 87%, LipidMaps 93%) in results_spotted.csv

## Limitations

- Adduct matching is exact-mass only and reports all possible isomers per m/z (level 4 Metabolomics Standards Initiative match), so identities are not uniquely resolved
- No explicit SpaMTP version number was recorded in the source; the installed version should be recorded by the user
- The paired targeted metabolomics refinement section was a placeholder ('To come!'), so adduct-based annotation cannot yet be cross-validated against targeted data within this pipeline

## Evidence

- [methods] Calculate 47 common adducts in positive and negative mode for each neutral mass entry and append them to the reference table.: [evidence span withheld — anchor status: not_found]
- [methods] AdductRules returns ion masses, multiplier, charge, proton-loss count, chemical class, prior: "`AdductRules()` returns the ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior used by the engine"
- [readme] SpaMTP README lists m/z metabolite annotation as first major functionality: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract states automated m/z metabolite annotation is pipeline step 1: "SpaMTP’s comprehensive analysis pipeline covers (1) automated mass-to-charge ratio (m/z) metabolite annotation;"
