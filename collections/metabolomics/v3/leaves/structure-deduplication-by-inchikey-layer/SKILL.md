---
name: structure-deduplication-by-inchikey-layer
description: Use when building a non-redundant structural dataset from spectral library
  entries (e.g. GNPS .mgf) where duplicate structures and stereoisomers must be collapsed
  to unique first-layer InChIKeys.
when_to_use_negative:
- Stereochemistry must be preserved as distinct candidates (e.g. benchmarking stereospecific
  annotation), since deduplication by the first 14 InChIKey characters merges stereoisomers
  by design.
- Spectra are still under the mgf pre-filter (fewer than 6 fragments or >500 peaks
  unprocessed) — apply mgf filtering first, as this deduplication operates on the
  cleaned entry set.
- The goal is detecting (near-)duplicate molecules across databases for clustering
  rather than building a benchmarking set — a broader similarity search is needed,
  not connectivity-layer dedup.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0219
tools:
- name: RDKit
  role: Converts each entry's structure to its 27-character InChIKey, from which the
    first 14 characters are extracted as the deduplication key
- name: mgf_filterer
  role: Upstream preprocessing that filters .mgf spectral files (≥6 fragments, 500
    most intense peaks) before parsing, cleaning and InChIKey-based deduplication
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:99aa94500ca0ee676f02c918c97ff3bc668ba4cc587253234a937c26b15d87cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/taxonomicallyinformedannotation/tima
  tools:
  - RDKit
  - mgf_filterer
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# structure-deduplication-by-inchikey-layer

## Summary

Collapses duplicate structures and stereoisomers in a spectral library dataset by comparing the first layer (first 14 characters) of the RDKit-generated InChIKey, keeping one entry per unique structural layer. Used in the taxonomically informed annotation pipeline to build the 40,138-entry benchmarking set from 8,558 unique structures.

## When to use

Use when building a non-redundant structural dataset from spectral library entries (e.g. GNPS .mgf) where duplicate structures and stereoisomers must be collapsed to unique first-layer InChIKeys.

## When NOT to use

- Stereochemistry must be preserved as distinct candidates (e.g. benchmarking stereospecific annotation), since deduplication by the first 14 InChIKey characters merges stereoisomers by design.
- Spectra are still under the mgf pre-filter (fewer than 6 fragments or >500 peaks unprocessed) — apply mgf filtering first, as this deduplication operates on the cleaned entry set.
- The goal is detecting (near-)duplicate molecules across databases for clustering rather than building a benchmarking set — a broader similarity search is needed, not connectivity-layer dedup.

## Inputs

- Cleaned GNPS spectral library entries (.mgf-derived, e.g. entries retained after mgf_filterer step) with parent masses within 100–1,500 Da and [M+H]+ adducts
- Structural identifiers (SMILES/InChI or similar) present for each entry
- RDKit-parseable structures for InChIKey generation

## Outputs

- Deduplicated spectral entry table (e.g. gnps_library_filtered.tsv) with one entry per unique first-layer InChIKey
- Set of 40,138 spectral entries over 8,558 unique first-layer InChIKeys in the article's application

## How to apply

Convert each retained library entry's structure to its InChIKey using RDKit — the 27-character hashed form of the full InChI — after cleaning metadata and filtering to protonated [M+H]+ entries with structural identifiers and parent masses between 100 and 1,500 Da. Deduplicate by keeping only distinct InChIKey values according to the first layer (first 14 characters), which collapses stereoisomers and duplicate connectivities while retaining one spectrum per structure. In the article's GNPS preprocessing this step, applied together with the ≥6-fragment / 500-most-intense-peaks mgf filter and 100–1,500 Da parent mass filter, reduced 66,646 concatenated GNPS library entries to 40,138 spectral entries covering 8,558 unique structures. The rationale is that structural annotation benchmarks should not inflate results by scoring multiple spectra of the same connectivity separately; a full-InChIKey comparison would keep stereoisomers distinct, so only the first (connectivity) layer is used.

## Related tools

- **RDKit** (Converts each entry's structure to its 27-character InChIKey, from which the first 14 characters are extracted as the deduplication key)
- **mgf_filterer** (Upstream preprocessing that filters .mgf spectral files (≥6 fragments, 500 most intense peaks) before parsing, cleaning and InChIKey-based deduplication) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
python scripts/parse_clean_gnps.py --input gnps_library.mgf --output gnps_library_filtered.tsv --dedup-key inchikey_first_layer
```

## Evaluation signals

- Resulting table contains exactly one entry per unique first-layer InChIKey — verify by counting distinct first 14 characters and confirming no duplicates remain.
- Article's reference outcome: 40,138 spectral entries over 8,558 unique first-layer InChIKeys when applied to the filtered GNPS libraries.
- All retained entries have parent masses within 100–1,500 Da and spectra satisfying the ≥6-fragment / ≤500-peak criteria.
- All InChIKeys are 27 characters, generated by RDKit from valid structures.
- Every retained entry has a structural identifier ([M+H]+ protonated adduct with a resolvable structure) — entries lacking identifiers should have been dropped upstream.

## Limitations

- Deduplication by the first 14 InChIKey characters intentionally removes stereoisomers, which is inappropriate when stereochemistry distinguishes candidates.
- The 100–1,500 Da parent mass window and [M+H]+ restriction bound applicability to protonated small-molecule entries only.
- GNPS spectral libraries were also part of the Sirius training set, confounding downstream benchmarking comparisons of annotation tools.
- A subset of entries (197 in the benchmarking dataset) had large mass differences (> 0.01 Da) between experimental parent ion mass and calculated exact mass — problematic entries are listed online (Problematic_entries.csv) but cannot be automatically identified by annotation tools.

## Evidence

- [discussion] Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters): "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] converting structures to their InChIKey, a 27-character hashed version of the full InChI: "converting structures to their InChIKey, a 27-character hashed version of the full InChI"
- [other] A pretreatment pipeline (mgf filtering plus parsing/cleaning) applied to the 66,646 concatenated GNPS library entries yielded a dataset of 40,138 structures (8,558 unique structures): "yielded a dataset of 40,138 structures (8,558 unique structures)"
- [discussion] filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [readme] README: tima — This repository contains everything needed to perform **T**axonomically **I**nformed **M**etabolite **A**nnotation.: "This repository contains everything needed to perform **T**axonomically **I**nformed **M**etabolite **A**nnotation."
