---
name: isomer-candidate-summarisation
description: Use when a spatial metabolomics peak list (e.g. exported from imzML)
  needs metabolite names assigned to detected m/z values without relying on METASPACE/pySM,
  and all isomeric candidates per peak must be preserved.
when_to_use_negative:
- When data come from high-resolution instruments (FT-ICR, orbitrap) where METASPACE/pySM's
  FDR-controlled annotation applies — SpaMTP was built because those tools 'do not
  yet apply to most data obtained with time-of-flight (TOF) analysers'
- When already-annotated data need downstream statistics (pathway analysis, differential
  expression) — use SpaMTP's CalculateAnnotationStatistics or AnnotateBigData refinement
  functions instead of re-annotating
- When confident structural ID beyond MSI level 4 is required — this step cannot disambiguate
  isomers without additional evidence (e.g. pseudo-MS/MS via MS1-ID/GNPS)
tools:
- name: SpaMTP
  role: Provides the automated m/z-to-metabolite annotation mechanism (AnnotateBigData)
    that performs exact-mass matching and isomer summarisation
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: R
  role: Implementation language in which SpaMTP's annotation module runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:b6edcb437d1b7163bea946015c8d64f6927ca181c289b9b0d23f432474b90c4d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - R
  - SpaMTP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# isomer-candidate-summarisation

## Summary

SpaMTP's platform-agnostic exact-mass m/z annotation step: peaks are matched against adduct-expanded compound databases and every possible isomer per matched m/z is reported, corresponding to a level 4 Metabolomics Standards Initiative annotation.

## When to use

Use when a spatial metabolomics peak list (e.g. exported from imzML) needs metabolite names assigned to detected m/z values without relying on METASPACE/pySM, and all isomeric candidates per peak must be preserved.

## When NOT to use

- When data come from high-resolution instruments (FT-ICR, orbitrap) where METASPACE/pySM's FDR-controlled annotation applies — SpaMTP was built because those tools 'do not yet apply to most data obtained with time-of-flight (TOF) analysers'
- When already-annotated data need downstream statistics (pathway analysis, differential expression) — use SpaMTP's CalculateAnnotationStatistics or AnnotateBigData refinement functions instead of re-annotating
- When confident structural ID beyond MSI level 4 is required — this step cannot disambiguate isomers without additional evidence (e.g. pseudo-MS/MS via MS1-ID/GNPS)

## Inputs

- Peak list of detected m/z values (e.g. from an imzML mouse liver MALDI dataset read via Cardinal)
- SDF-derived compound databases: ChEBI, HMDB, GNPS, LIPIDMAPS
- User-defined polarity, adduct set, and ppm error tolerance (±3 ppm benchmarked)

## Outputs

- Per-peak annotation table (e.g. annotation_test_results.csv) with exact mass, formula, annotation name, InChIKey, ppm difference, and adduct type per match
- All possible isomer candidates per matched m/z (level 4 MSI annotation)

## How to apply

Convert the downloaded SDF compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS) into a uniform reference table, removing N/A and zero-value entries. Calculate 47 common adducts in positive and negative mode for each neutral mass entry and append them to the reference table, then filter by user-defined polarity, adduct set, and an elemental-composition constraint limited to common natural-product elements (C, H, N, S, O, P, Br, F, Na, I, Si). Perform exact-mass matching of every valid m/z against the filtered database at a user-defined ppm error (±3 ppm as benchmarked on the human lung data). Summarise results by reporting all possible isomers per matched m/z — deliberately a level 4 MSI match rather than a single identity — and validate concordance against the deposited reference table (results_spotted.csv), consistent with the reported ~90% name agreement with METASPACE (HMDB 87%, LipidMaps 93%).

## Related tools

- **SpaMTP** (Provides the automated m/z-to-metabolite annotation mechanism (AnnotateBigData) that performs exact-mass matching and isomer summarisation) — https://github.com/GenomicsMachineLearning/SpaMTP
- **R** (Implementation language in which SpaMTP's annotation module runs)

## Examples

```
if (!require("SpaMTP")) devtools::install_github("GenomicsMachineLearning/SpaMTP")
```

## Evaluation signals

- Every annotated peak record carries all seven expected fields: exact mass, formula, annotation name, InChIKey, ppm difference, and adduct type
- All m/z matches fall within the user-defined ppm error (±3 ppm in the benchmark)
- Each matched m/z reports every possible isomer, not a single best hit (MSI level 4 semantics)
- Concordance against the deposited reference table (results_spotted.csv) reproduces the reported ~90% name agreement with METASPACE (HMDB 87%, LipidMaps 93%)
- Reference table preprocessing removed all N/A and zero-value entries before adduct expansion

## Limitations

- Isomer-level ambiguity is unresolved: results are level 4 Metabolomics Standards Initiative matches, requiring pseudo-MS/MS evidence (MS1-ID/GNPS spectral database) for higher confidence
- Elemental-composition constraint excludes compounds containing elements outside C, H, N, S, O, P, Br, F, Na, I, Si
- Annotation accuracy depends on the compound databases used (HMDB 87%, LipidMaps 93% name agreement with METASPACE, i.e. not perfect)

## Evidence

- [methods] Summarise isomers by reporting all possible isomers per matched m/z, corresponding to a level 4 Metabolomics Standards Initiative match.: [evidence span withheld — anchor status: not_found]
- [methods] returning exact mass, formula, annotation name, InChIKey, ppm difference, and adduct type for each match: [evidence span withheld — anchor status: not_found]
- [intro] METASPACE and pySM are optimised for high-resolution instrument data and do not yet apply to most data obtained with time-of-flight (TOF) analysers: [evidence span withheld — anchor status: not_found]
- [readme] mass-to-charge ratio (m/z) metabolite annotation, (2) various downstream statistical analysis including differential metabolite expression and pathway analysis: [evidence span withheld — anchor status: external_doc]
