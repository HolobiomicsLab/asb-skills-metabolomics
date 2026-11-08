---
name: exact-mass-metabolite-annotation
description: Use when spatial metabolomics peaks (e.g. 401,325 m/z values from an
  imzML MALDI-TOF dataset) need metabolite name, formula, InChIKey and adduct assignments
  without METASPACE/pySM, especially for TOF data.
when_to_use_negative:
- When the goal is FDR-controlled high-resolution annotation — METASPACE/pySM is designed
  for automated annotation based on FDR and is optimised for FT-ICR/orbitrap data,
  though it does not yet apply to most TOF data
- When structural confirmation beyond MS1 exact mass is required — this method yields
  level 4 MSI isomer sets, not validated structures
- Input is already an annotated feature table; use downstream steps (RefineLipids,
  CalculateAnnotationStatistics) instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaMTP
  role: Provides the annotation module (AnnotateSM / AnnotateBigData) that converts
    SDF databases to a normalised reference, expands adducts, filters by polarity/elements,
    and matches m/z at ±3 ppm
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Cardinal
  role: Reads the imzML spatial metabolomics files via SpaMTP's LoadSM to supply the
    detected m/z values for annotation
- name: METASPACE
  role: External FDR-based annotation tool used as an independent comparison benchmark
    (~87% HMDB, 93% LIPIDMAPS name agreement)
- name: HMDB
  role: Source compound database converted to the internal reference format and searched
    in the polarity/adduct space
- name: LIPIDMAPS
  role: Source lipid compound database converted to the internal reference format
    and searched in the polarity/adduct space
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:9a925ec7d88e7818908bd7acface424eb14e8fd119e15329afc7637f5cc97e84
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - HMDB
  - LIPIDMAPS
  - METASPACE
  - R
  - SpaMTP
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# exact-mass-metabolite-annotation

## Summary

Assigns metabolite identities to detected m/z peaks by exact-mass matching (±3 ppm benchmarked) against pre-compiled compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS) with polarity/adduct filtering, using SpaMTP's annotation module. It is platform-agnostic, working on data from any instrument including TOF analysers where METASPACE/pySM underperform.

## When to use

Use when spatial metabolomics peaks (e.g. 401,325 m/z values from an imzML MALDI-TOF dataset) need metabolite name, formula, InChIKey and adduct assignments without METASPACE/pySM, especially for TOF data.

## When NOT to use

- When the goal is FDR-controlled high-resolution annotation — METASPACE/pySM is designed for automated annotation based on FDR and is optimised for FT-ICR/orbitrap data, though it does not yet apply to most TOF data
- When structural confirmation beyond MS1 exact mass is required — this method yields level 4 MSI isomer sets, not validated structures
- Input is already an annotated feature table; use downstream steps (RefineLipids, CalculateAnnotationStatistics) instead

## Inputs

- Detected m/z peak list from spatial metabolomics data (e.g. imzML read via Cardinal/LoadSM; human lung MALDI-TOF with 401,325 m/z values)
- SDF compound database files (ChEBI, HMDB, GNPS, LIPIDMAPS)
- User-defined polarity (positive/negative) and adduct set

## Outputs

- Per-peak annotation table with exact mass, formula, annotation name, InChIKey, ppm difference, and adduct type (e.g. annotation_test_results.csv)
- All possible isomers per matched m/z (level 4 MSI annotation)

## How to apply

First convert the downloaded SDF compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS) into a uniform reference table, removing N/A and zero-value entries. Compute 47 common adducts in positive and negative mode for each neutral mass and append them to the table, then filter by user-defined polarity, adduct set, and an elemental-composition constraint limited to C, H, N, S, O, P, Br, F, Na, I, Si. Perform exact-mass matching of every valid m/z against the filtered database at a user-defined ppm error (±3 ppm benchmarked on the human lung data), returning exact mass, formula, annotation name, InChIKey, ppm difference and adduct type per match. Report all isomers per matched m/z, corresponding to a level 4 Metabolomics Standards Initiative match; for large peak lists, SpaMTP's BuildMZAnnotationIndex() performs the expensive expansion once so QueryMZAnnotationIndex() can use binary interval search.

## Related tools

- **SpaMTP** (Provides the annotation module (AnnotateSM / AnnotateBigData) that converts SDF databases to a normalised reference, expands adducts, filters by polarity/elements, and matches m/z at ±3 ppm) — github.com/GenomicsMachineLearning/SpaMTP
- **Cardinal** (Reads the imzML spatial metabolomics files via SpaMTP's LoadSM to supply the detected m/z values for annotation)
- **METASPACE** (External FDR-based annotation tool used as an independent comparison benchmark (~87% HMDB, 93% LIPIDMAPS name agreement))
- **HMDB** (Source compound database converted to the internal reference format and searched in the polarity/adduct space)
- **LIPIDMAPS** (Source lipid compound database converted to the internal reference format and searched in the polarity/adduct space)

## Examples

```
Rscript -e 'devtools::install_github("GenomicsMachineLearning/SpaMTP"); lung <- SpaMTP::LoadSM("human_lung_imzml"); annotated <- SpaMTP::AnnotateSM(lung, dbs=c("HMDB","LIPIDMAPS"), ppm=3)'
```

## Evaluation signals

- Annotation concordance against the deposited reference table (results_spotted.csv) from the mouse liver spotted-standards dataset matches the reported ~90% name agreement with METASPACE (HMDB 87%, LIPIDMAPS 93%)
- Per-database annotation counts on the human lung data reproduce the expected scale (SpaMTP ~80,235 m/z for HMDB and ~39,215 for LIPIDMAPS vs METASPACE 247 and 322)
- Every reported match includes ppm difference within the user-defined tolerance (±3 ppm) and a valid adduct type
- Isomer groups are fully enumerated per matched m/z rather than collapsed to a single name
- Reference table contains no N/A or zero-value mass entries after normalisation, and only elements C, H, N, S, O, P, Br, F, Na, I, Si pass the filter

## Limitations

- Exact-mass matching alone cannot resolve isomers; all isomers are reported, corresponding to a level 4 Metabolomics Standards Initiative match
- The elemental-composition constraint restricts candidates to common natural-product elements (C, H, N, S, O, P, Br, F, Na, I, Si), excluding metabolites containing other elements
- METASPACE/pySM annotations are FDR-controlled whereas SpaMTP's exact-mass matching is not, so raw annotation counts are not directly comparable without reconciliation of synonyms
- Paired targeted metabolomics comparison for refinement is a placeholder ('To come!') in the vignette
- No explicit SpaMTP version number was recorded in the source article

## Evidence

- [methods] Convert SDF databases and match m/z at ±3 ppm: [evidence span withheld — anchor status: not_found]
- [results] SpaMTP annotates agnostic of the data-generating platform: [evidence span withheld — anchor status: not_found]
