---
name: dnp-structure-source-dataset-harmonization
description: Use when building a benchmark or reference set that pairs MS/MS spectra
  (.mgf) with organism-source metadata from the Dictionary of Natural Products, requiring
  structure harmonization via short InChIKey and taxonomy filtering.
when_to_use_negative:
- Your goal is per-sample species-specific source resolution for annotation scoring
  rather than constructing the structure–spectra–source reference set
- Spectral entries lack InChIKey-decodable structures or have unresolved taxonomy
  beyond species level — the join cannot retain them
- Input spectra have fewer than 6 fragments or parent masses outside 100–1,500 Da,
  as those were already filtered out upstream of this harmonization
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3346
- http://edamontology.org/topic_0219
tools:
- name: gnfinder
  role: Detects taxonomic names in free-text source fields and matches them against
    Global Names index services so entries can be filtered to Catalogue-of-Life-resolved,
    species-level sources
  repo: https://github.com/gnames/gnfinder
- name: RDKit
  role: Converts DNP structures to InChIKey; the first 14 characters are used to harmonize
    structures across the DNP and GNPS datasets and deduplicate stereoisomers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:98c07b10a5241b5c4d18c7b1003d2e31da0750d96bc5b718aa6a724e799abef2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - RDKit
  - gnfinder
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3346
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# dnp-structure-source-dataset-harmonization

## Summary

Harmonizes the curated DNP-derived structure–biological source table with the filtered GNPS spectral dataset (40,138 entries, 8,558 unique structures) by matching on the short (first 14 characters) InChIKey, deduplicating stereoisomers, and retaining only Catalogue-of-Life-resolved species-level sources to build the paper's 2,107-entry benchmarking dataset.

## When to use

Use when building a benchmark or reference set that pairs MS/MS spectra (.mgf) with organism-source metadata from the Dictionary of Natural Products, requiring structure harmonization via short InChIKey and taxonomy filtering.

## When NOT to use

- Your goal is per-sample species-specific source resolution for annotation scoring rather than constructing the structure–spectra–source reference set
- Spectral entries lack InChIKey-decodable structures or have unresolved taxonomy beyond species level — the join cannot retain them
- Input spectra have fewer than 6 fragments or parent masses outside 100–1,500 Da, as those were already filtered out upstream of this harmonization

## Inputs

- Curated DNP-derived table of structures with associated biological-source metadata (DNP v 27.1)
- Filtered GNPS-library spectral dataset (.mgf format; 40,138 entries, 8,558 unique structures)
- Structures convertible to InChIKey via RDKit
- Catalogue of Life taxonomy backbone for name resolution

## Outputs

- Benchmarking dataset of 2,107 distinct structure–spectra–source entries
- Spectral side of the benchmarking dataset in .mgf format
- Associated sample/source metadata in .tsv format

## How to apply

Load the curated DNP-derived structure–biological-source table and the filtered GNPS-library spectral dataset (40,138 entries covering 8,558 unique structures) from the OSF deposit (https://osf.io/bvs6x/). Harmonize both tables by the short InChIKey — the first 14 characters of the 27-character InChIKey — so structures match across the structure and spectral sides, and remove duplicate structures and stereoisomers by keeping distinct entries per this first InChIKey layer. Join on the short InChIKey, retaining only structure–source associations whose taxonomy was resolved against the Catalogue of Life (via gnfinder matching against the Global Names index) and complete down to the species level. Write the spectral side to an .mgf file and the associated sample/source metadata to a .tsv file. Validate that the merged output contains exactly 2,107 distinct entries — the paper's reported benchmarking dataset size — before using it for taxonomically informed scoring benchmarks.

## Related tools

- **gnfinder** (Detects taxonomic names in free-text source fields and matches them against Global Names index services so entries can be filtered to Catalogue-of-Life-resolved, species-level sources) — https://github.com/gnames/gnfinder
- **RDKit** (Converts DNP structures to InChIKey; the first 14 characters are used to harmonize structures across the DNP and GNPS datasets and deduplicate stereoisomers)

## Examples

```
Rscript 1_taxo_cleaner.Rmd  # resolve DNP/GNPS structures by short InChIKey and Catalogue of Life taxonomy
```

## Evaluation signals

- Merged output contains exactly 2,107 distinct entries, matching the paper's reported benchmarking dataset size
- Both output files (.mgf and .tsv) exist and are well-formed
- Every retained entry's biological source is resolved against the Catalogue of Life and complete down to the species level
- No two retained entries share the same first-14-character InChIKey (stereoisomer/duplicate removal verified)
- Parent masses of retained spectra fall within 100–1,500 Da and each spectrum has at least 6 fragments

## Limitations

- Only entries with taxonomy resolved against Catalogue of Life and complete to species level are retained; unresolved or higher-level taxa are dropped
- All compounds of the resulting benchmarking dataset are present in the DNP, which biases downstream tool comparisons in favor of ISDB-DNP
- 197 benchmarking entries had large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass, listed online as Problematic_entries.csv
- The DNP provides taxonomic information in free-text fields, requiring gnfinder extraction rather than structured lookup

## Evidence

- [discussion] benchmarking dataset of 2,107 entries: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [discussion] short InChIKey deduplication: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] Catalogue of Life species-level filter: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] OSF data deposit: "Input and output data are available on OSF at the following address (https://osf.io/bvs6x/)"
- [discussion] gnfinder name resolution: "gnfinder allows to match the detected names against the Global Names index services"
- [discussion] mgf/tsv output formats: "created to filter.mgf spectral file according to specific parameters"
- [discussion] InChIKey conversion via RDKit: "converting structures to their InChIKey, a 27-character hashed version of the full InChI"
- [discussion] free-text taxonomy in DNP: "In the Dictionary of Natural Products (v 27.1), taxonomic information appears in two fields"
