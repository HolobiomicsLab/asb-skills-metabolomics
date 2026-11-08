---
name: reference-library-count-validation
description: Use when a GNPS GC-MS library search will be run against the freely available
  released reference library and the spectra/standards counts must be verified as
  19,808 spectra for 19,708 standards before matching deconvoluted .mgf features.
when_to_use_negative:
- Input is LC-MS/MS data intended for molecular networking workflows (e.g. pyMolNetEnhancer
  chemical-class mapping) rather than deconvoluted GC-MS spectra
- The deconvoluted .mgf has not yet been produced by the upstream MSHub/GNPS auto-deconvolution
  step
- The library will be a user-supplied or public non-manuscript library, where the
  19,808/19,708 count invariants do not apply
tools:
- name: MSHub
  role: Machine-learning auto-deconvolution workflow that produces the deconvoluted
    compound fragmentation spectra matched against the validated GC-MS reference library
  repo: https://github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform whose GC-MS library search workflow consumes the validated reference
    library to score matches via cosine similarity on fragment ions
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:e83980d9f11bdd5e81a2d25d20891f1e0ecd3aa9efc8ba2424f09bf2b3a65bf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GNPS_GC
  tools:
  - GNPS
  - MSHub
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# reference-library-count-validation

## Summary

Validates that the GNPS GC-MS reference library released with the Aksenov et al. (2020) MSHub manuscript parses to exactly 19,808 spectra covering 19,708 unique standards before using it in the GNPS GC-MS library search. This guards downstream annotation of deconvoluted .mgf spectra against an incomplete or malformed reference set.

## When to use

Use when a GNPS GC-MS library search will be run against the freely available released reference library and the spectra/standards counts must be verified as 19,808 spectra for 19,708 standards before matching deconvoluted .mgf features.

## When NOT to use

- Input is LC-MS/MS data intended for molecular networking workflows (e.g. pyMolNetEnhancer chemical-class mapping) rather than deconvoluted GC-MS spectra
- The deconvoluted .mgf has not yet been produced by the upstream MSHub/GNPS auto-deconvolution step
- The library will be a user-supplied or public non-manuscript library, where the 19,808/19,708 count invariants do not apply

## Inputs

- Released GNPS GC-MS reference library (19,808 spectra / 19,708 standards) from the manuscript companion data
- Deconvoluted query spectra in .mgf format from GNPS-MSHub auto-deconvolution (for downstream use, not for the count check)

## Outputs

- Validated library object with confirmed spectra count (19,808) and unique-standards count (19,708)
- library_search_annotations.tsv mapping each query spectrum to its best matching library standard with match scores

## How to apply

Download the GNPS GC-MS reference library released with the manuscript (companion data of doi:10.1038/s41587-020-0700-3, repo bittremieux/GNPS_GC) and parse its spectra into individual entries. Count total spectra and unique standards, asserting exactly 19,808 spectra and 19,708 unique standards; a mismatch indicates a truncated download or a parsing bug in the spectral reader. Confirm each parsed entry carries the fragment-ion peaks needed for cosine-similarity scoring in the GNPS GC-MS library search, and retains any Kovats retention-index annotations used later to restrict candidate matches. Only after validation should the parsed library be registered as the reference for matching the deconvoluted spectra from the MSHub/GNPS auto-deconvolution .mgf output, with best matches written to library_search_annotations.tsv.

## Related tools

- **MSHub** (Machine-learning auto-deconvolution workflow that produces the deconvoluted compound fragmentation spectra matched against the validated GC-MS reference library) — https://github.com/bittremieux/GNPS_GC
- **GNPS** (Platform whose GC-MS library search workflow consumes the validated reference library to score matches via cosine similarity on fragment ions) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Evaluation signals

- Parsed spectra count is exactly 19,808 and unique-standard count is exactly 19,708
- Every parsed library entry contains non-empty peak lists usable for cosine-similarity fragment-ion matching
- Each library standard maps to at most one unique identifier with no duplicate spectra loss during parsing
- Retention-index annotations, where present, remain attached to the correct standards so RI-restricted candidate matching can proceed
- Post-search, library_search_annotations.tsv contains one best matching standard per query spectrum with a finite match score

## Limitations

- The count invariant (19,808 spectra / 19,708 standards) applies only to the specific library release accompanying the manuscript; future library versions will differ
- Cosine-similarity scoring alone can yield ambiguous matches; restricting candidates by Kovats retention index is only possible where such annotations are available
- The bittremieux/GNPS_GC companion repository is archived, so the released library snapshot may not receive updates

## Evidence

- [other] The GNPS library search can use the freely available reference data released with the manuscript, which comprises 19,808 spectra for 19,708 standards,: [evidence span withheld — anchor status: not_found]
