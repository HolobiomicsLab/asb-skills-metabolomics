---
name: spatial-metabolomics-annotation-benchmarking
description: Use when comparing an annotation engine's coverage against a published
  METASPACE/pySM baseline on MALDI imaging data, e.g. counting m/z values annotated
  against HMDB and LipidMaps and measuring name-match fractions.
when_to_use_negative:
- Input data was generated on a high-resolution instrument (FT-ICR or Orbitrap) where
  METASPACE/pySM itself is optimised and applicable — a SpaMTP-vs-METASPACE comparison
  adds little
- The goal is structural (MS/MS-based) annotation rather than exact-mass MS1 m/z annotation
  — use the MS1-ID/GNPS pseudo-MS/MS workflow instead
- No published METASPACE annotations exist for the dataset, so the baseline side of
  the benchmark cannot be assembled
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0121
tools:
- name: SpaMTP
  role: R package providing LoadSM data import, AnnotateSM annotation pipeline, and
    m/z metabolite annotation agnostic of the data-generating platform
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Cardinal
  role: Underlying imzML reader wrapped by SpaMTP's LoadSM for loading MALDI imaging
    data
- name: METASPACE
  role: 'Baseline annotation source: published FDR-20% annotations for HMDB and LipidMaps
    against which SpaMTP output is compared'
- name: HMDB
  role: Reference metabolite database searched by both engines (expected ~80,235 SpaMTP
    annotations vs 247 METASPACE, 87% name match)
- name: LIPIDMAPS
  role: Reference lipid database searched by both engines (expected ~39,215 SpaMTP
    annotations vs 322 METASPACE, 93% name match)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:ffedb4c3c8053568571d9eda7ac52918122e92f32b75dab06451d7a1da3f51b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - HMDB
  - LIPIDMAPS
  - METASPACE
  - SpaMTP
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spatial-metabolomics-annotation-benchmarking

## Summary

Benchmark SpaMTP's platform-agnostic m/z metabolite annotation against METASPACE/pySM annotations on the same spatial metabolomics dataset, quantifying per-database annotation counts and metabolite-name agreement among shared masses.

## When to use

Use when comparing an annotation engine's coverage against a published METASPACE/pySM baseline on MALDI imaging data, e.g. counting m/z values annotated against HMDB and LipidMaps and measuring name-match fractions.

## When NOT to use

- Input data was generated on a high-resolution instrument (FT-ICR or Orbitrap) where METASPACE/pySM itself is optimised and applicable — a SpaMTP-vs-METASPACE comparison adds little
- The goal is structural (MS/MS-based) annotation rather than exact-mass MS1 m/z annotation — use the MS1-ID/GNPS pseudo-MS/MS workflow instead
- No published METASPACE annotations exist for the dataset, so the baseline side of the benchmark cannot be assembled

## Inputs

- MALDI-TOF imaging dataset in imzML/ibd format (human lung), read via SpaMTP LoadSM / Cardinal readImzML
- HMDB and LipidMaps reference databases converted to SpaMTP's normalised internal reference format
- Published METASPACE annotation results for the same dataset (max FDR 20%, HMDB and LipidMaps)

## Outputs

- Per-database counts of annotated m/z values for SpaMTP and METASPACE (HMDB, LipidMaps)
- Fraction of matching metabolite names among shared m/z masses (synonyms reconciled manually)
- Benchmark table written to lung_annotation_benchmark.csv

## How to apply

Load the MALDI-TOF spatial metabolomics dataset via SpaMTP's LoadSM (wrapping Cardinal's imzML reader) and confirm the full set of detected m/z values (401,325 for the human lung dataset). Run SpaMTP's AnnotateSM pipeline on all valid m/z values with a ±3 ppm exact-mass error tolerance, searching HMDB and LipidMaps converted to the normalised internal reference format in the appropriate polarity/adduct space. Retrieve published METASPACE annotations for the same dataset at a maximum FDR of 20% for both databases, then match overlapping m/z masses between the two engines, manually reconciling synonyms of the same metabolite. Per database, record the number of annotated m/z values (expected ~80,235 for HMDB and ~39,215 for LipidMaps via SpaMTP vs 247 and 322 via METASPACE) and the fraction of matching metabolite names among shared masses (expected ~90%: HMDB 87%, LipidMaps 93%). Assemble counts and match percentages into a single benchmark table written to lung_annotation_benchmark.csv.

## Related tools

- **SpaMTP** (R package providing LoadSM data import, AnnotateSM annotation pipeline, and m/z metabolite annotation agnostic of the data-generating platform) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Cardinal** (Underlying imzML reader wrapped by SpaMTP's LoadSM for loading MALDI imaging data)
- **METASPACE** (Baseline annotation source: published FDR-20% annotations for HMDB and LipidMaps against which SpaMTP output is compared)
- **HMDB** (Reference metabolite database searched by both engines (expected ~80,235 SpaMTP annotations vs 247 METASPACE, 87% name match))
- **LIPIDMAPS** (Reference lipid database searched by both engines (expected ~39,215 SpaMTP annotations vs 322 METASPACE, 93% name match))

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP"); lung <- LoadSM("./human_lung_maldi_tof"); annotated <- AnnotateSM(lung, databases = c("HMDB", "LipidMaps"), ppm_error = 3)
```

## Evaluation signals

- All 401,325 detected m/z values confirmed present after LoadSM import of the human lung MALDI-TOF imzML dataset
- SpaMTP annotation counts approximate the expected ~80,235 (HMDB) and ~39,215 (LipidMaps) m/z values at the ±3 ppm tolerance
- METASPACE baseline counts equal the published 247 (HMDB) and 322 (LipidMaps) at max FDR 20%
- Name-match fractions among shared masses fall near the expected ~90% (HMDB 87%, LipidMaps 93%) after manual synonym reconciliation
- lung_annotation_benchmark.csv contains one row per database with SpaMTP count, METASPACE count, and match percentage columns populated

## Limitations

- METASPACE and pySM are optimised for high-resolution instrument data (FT-ICR, Orbitrap) and do not yet apply to most TOF data, so the baseline itself is platform-biased
- Synonyms of the same metabolite must be reconciled manually during name matching, introducing subjective judgement into the match percentage
- Exact-mass annotation at ±3 ppm is a one-dimensional match; SpaMTP's refinement options (CalculateAnnotationStatistics pathway prioritisation, RefineLipids, MS1-ID pseudo-MS/MS evidence) are not part of this raw benchmark
- No explicit SpaMTP version number is stated for the benchmark, so exact reproducibility of counts may vary across versions

## Evidence

- [intro] Platform-agnostic annotation claim: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline coverage: [evidence span withheld — anchor status: not_found]
