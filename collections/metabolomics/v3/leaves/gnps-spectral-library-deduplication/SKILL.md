---
name: gnps-spectral-library-deduplication
description: Use when building a reference/benchmarking set from GNPS spectral libraries
  whose entries must map one-to-one to unique structures (no stereoisomer distinction),
  before joining spectra to structure–biological-source tables.
when_to_use_negative:
- When stereoisomer-level resolution matters — this convention deliberately collapses
  entries sharing the first 14 InChIKey characters, so isomeric structures are merged.
- When the goal is spectral-quality filtering (fragment counts, parent mass windows,
  peak-intensity caps) rather than structural deduplication; those filters (≥6 fragments,
  100–1,500 Da) are upstream steps, not a substitute for this skill.
- When working with raw acquired sample spectra rather than library spectra with resolved
  structures and InChIKeys.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RDKit
  role: Converts library structures to InChIKeys; the first 14 characters provide
    the deduplication key that removes duplicate structures and stereoisomers.
- name: R / data.table
  role: Harmonizes the filtered GNPS spectral table and the DNP-derived source table
    on the short InChIKey, merges them, and writes the .mgf/.tsv benchmarking outputs.
- name: GNPS spectral libraries
  role: Source of library spectra and structures to be deduplicated into the benchmarking
    dataset.
  repo: https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp
- name: tima
  role: Maintained repository containing the pipeline in which this deduplication/joining
    workflow is implemented.
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:b249b6e3610a89f503c7b1928f85d1e8335e9bed422c015f81e11d1cedb20160
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS spectral libraries
  - R / data.table
  - RDKit
  - gnfinder
  - tima
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# GNPS spectral library deduplication by short InChIKey

## Summary

Deduplicates merged GNPS spectral library entries by keeping distinct InChIKeys at the first structural layer (first 14 characters), collapsing duplicate structures and stereoisomers so each spectrum corresponds to a unique structure. This produced the paper's benchmarking dataset of 2,107 entries.

## When to use

Use when building a reference/benchmarking set from GNPS spectral libraries whose entries must map one-to-one to unique structures (no stereoisomer distinction), before joining spectra to structure–biological-source tables.

## When NOT to use

- When stereoisomer-level resolution matters — this convention deliberately collapses entries sharing the first 14 InChIKey characters, so isomeric structures are merged.
- When the goal is spectral-quality filtering (fragment counts, parent mass windows, peak-intensity caps) rather than structural deduplication; those filters (≥6 fragments, 100–1,500 Da) are upstream steps, not a substitute for this skill.
- When working with raw acquired sample spectra rather than library spectra with resolved structures and InChIKeys.

## Inputs

- GNPS spectral libraries and publicly accessible third-party libraries (downloaded from https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp)
- Filtered GNPS spectral dataset (e.g. 40,138 entries covering 8,558 unique structures from the OSF deposit)
- Curated DNP-derived structure–biological-source table with taxonomy resolved against the Catalogue of Life
- Structure representations (e.g. SMILES/InChI) per library entry for InChIKey computation via RDKit

## Outputs

- Deduplicated benchmarking dataset of 2,107 distinct entries (unique structures, no stereoisomer distinction, one biological source each)
- Spectral side written as an .mgf file
- Associated sample/source metadata written as a .tsv file

## How to apply

Retrieve all GNPS and publicly accessible third-party spectral libraries, then filter them first (retain entries with ≥6 fragments, cap at the 500 most intense peaks, keep parent masses between 100 and 1,500 Da). Convert each entry's structure to its InChIKey and truncate to the first 14 characters — the paper's convention for collapsing duplicate structures and stereoisomers, since the first InChIKey layer encodes the connectivity skeleton but not stereochemistry. Harmonize the spectral table with the curated structure–biological-source table (DNP-derived) on this short InChIKey, join the two tables, and retain only associations whose taxonomy is resolved against the Catalogue of Life and complete down to the species level. Write the spectral side as an .mgf file and the associated source metadata as a .tsv file; the correctly deduplicated join yields exactly 2,107 distinct entries.

## Related tools

- **RDKit** (Converts library structures to InChIKeys; the first 14 characters provide the deduplication key that removes duplicate structures and stereoisomers.)
- **R / data.table** (Harmonizes the filtered GNPS spectral table and the DNP-derived source table on the short InChIKey, merges them, and writes the .mgf/.tsv benchmarking outputs.)
- **GNPS spectral libraries** (Source of library spectra and structures to be deduplicated into the benchmarking dataset.) — https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp
- **tima** (Maintained repository containing the pipeline in which this deduplication/joining workflow is implemented.) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
setkey(spectra, short_inchikey); setkey(sources, short_inchikey); bench <- sources[spectra, nomatch = 0]; stopifnot(length(unique(bench$short_inchikey)) == 2107)
```

## Evaluation signals

- The final joined dataset contains exactly 2,107 distinct entries, matching the reported size of the paper's benchmarking dataset.
- No duplicated first-14-character InChIKeys remain — the count of unique short InChIKeys equals the number of retained entries.
- Both output files (.mgf spectra and .tsv source metadata) exist and are well-formed, with entries aligned one-to-one across the two files.
- Every retained entry's biological source is resolved against the Catalogue of Life and complete down to the species level.
- Entries surviving deduplication also satisfy the upstream filters (≥6 fragments, parent mass 100–1,500 Da, ≤500 most intense peaks).

## Limitations

- Collapsing on the first 14 InChIKey characters erases stereoisomer distinction by design, so stereochemically distinct natural products cannot be separated downstream.
- 197 benchmarking entries were found to have large mass differences (> 0.01 Da) between experimental parent ion mass and calculated exact mass; the problematic entries are listed online (Problematic_entries.csv) but cannot be identified by annotation tools.
- Because all compounds in the benchmarking dataset are present in the DNP, the ISDB-DNP tool is favored in downstream benchmarking, and the GNPS spectral libraries were also part of the Sirius training set — comparisons across tools are confounded.

## Evidence

- [discussion] Duplicate structures and stereoisomers removed by short InChIKey: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] Benchmarking dataset size of 2,107 entries: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [discussion] Taxonomic resolution filter applied to retained entries: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] Parent mass window applied to library entries before deduplication: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] Fragment count and peak-intensity caps on library spectra: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
