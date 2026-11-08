---
name: mgf-and-tsv-benchmark-dataset-export
description: 'Use when you need to construct the ground-truth benchmark set for evaluating
  MS/MS annotation tools: you have filtered GNPS library spectra plus a structure-organism
  source table and must produce paired .mgf and .tsv files.'
when_to_use_negative:
- Your goal is annotating unknown experimental features from samples rather than building
  the ground-truth library benchmark from GNPS/DNP data
- Your structure-source associations are not resolved against the Catalogue of Life
  or lack species-level taxonomy — the paper explicitly retains only species-complete
  entries
- The OSF deposit (osf.io/bvs6x) already provides the finalized 2,107-entry benchmarking
  dataset and you do not need to rebuild it from source tables
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: tima
  role: R repository providing the pipeline scripts (and Docker image adafede/tima-r)
    used to harmonize, filter, and export the benchmarking dataset files
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: gnfinder
  role: Extracts and verifies biological source names from free text against the Global
    Names index / Catalogue of Life before the join
  repo: https://github.com/gnames/gnfinder
- name: MsBackendMgf
  role: R backend for writing/reading the .mgf spectral export file
  repo: https://github.com/rformassspectrometry/MsBackendMgf
- name: R / data.table
  role: Used for the InChIKey-based merge, deduplication filtering, and .tsv export
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:0a2f332200312b67b6524ccbe4d5ae587e080b9b6a2ea31935f1e97c7226cb2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/taxonomicallyinformedannotation/tima
  tools:
  - MsBackendMgf
  - R / data.table
  - gnfinder
  - tima
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mgf-and-tsv-benchmark-dataset-export

## Summary

Builds and exports the taxonomically annotated benchmarking spectral dataset by joining GNPS spectral library entries with DNP-derived structure-biological source associations on the short (first 14-character) InChIKey, yielding the 2,107-entry benchmarking dataset used to score taxonomically informed annotation.

## When to use

Use when you need to construct the ground-truth benchmark set for evaluating MS/MS annotation tools: you have filtered GNPS library spectra plus a structure-organism source table and must produce paired .mgf and .tsv files.

## When NOT to use

- Your goal is annotating unknown experimental features from samples rather than building the ground-truth library benchmark from GNPS/DNP data
- Your structure-source associations are not resolved against the Catalogue of Life or lack species-level taxonomy — the paper explicitly retains only species-complete entries
- The OSF deposit (osf.io/bvs6x) already provides the finalized 2,107-entry benchmarking dataset and you do not need to rebuild it from source tables

## Inputs

- GNPS spectral library entries (filtered .mgf or equivalent, 40,138 entries / 8,558 unique structures; parent masses 100-1500 Da, at least 6 fragments, 500 most intense peaks)
- Curated DNP-derived structure-biological source table (Dictionary of Natural Products v27.1, taxonomy resolved against Catalogue of Life)
- Full InChIKey strings for each structure (to derive the 14-character short InChIKey)

## Outputs

- Benchmarking dataset .mgf file (2,107 spectra, [M+H]+ adducts, 500 most intense peaks per spectrum)
- Benchmarking dataset .tsv file with one unique biological source and resolved taxonomy per entry

## How to apply

Load the curated DNP-derived structure-biological source table and the filtered GNPS spectral dataset (40,138 entries covering 8,558 unique structures) from the OSF deposit (osf.io/bvs6x). Harmonize both tables by the short InChIKey (first 14 characters) so structures match across the structure and spectral sides, then join them keeping only associations whose biological source was resolved against the Catalogue of Life and is complete down to the species level. Apply the paper's deduplication convention — remove duplicate structures and stereoisomers by keeping distinct short InChIKeys — and remove the known 197 problematic entries with parent mass differences >0.01 Da if strict mass consistency is required. Write the spectral side to an .mgf file and the associated sample/source metadata to a .tsv file using R/data.table (or the tima repository scripts / Docker image adafede/tima-r). The output must contain exactly 2,107 distinct entries, matching the reported benchmarking dataset size.

## Related tools

- **tima** (R repository providing the pipeline scripts (and Docker image adafede/tima-r) used to harmonize, filter, and export the benchmarking dataset files) — https://github.com/taxonomicallyinformedannotation/tima
- **gnfinder** (Extracts and verifies biological source names from free text against the Global Names index / Catalogue of Life before the join) — https://github.com/gnames/gnfinder
- **MsBackendMgf** (R backend for writing/reading the .mgf spectral export file) — https://github.com/rformassspectrometry/MsBackendMgf
- **R / data.table** (Used for the InChIKey-based merge, deduplication filtering, and .tsv export)

## Evaluation signals

- Output contains exactly 2,107 distinct short InChIKeys (the reported benchmarking dataset size); any other count indicates a join or deduplication error
- Every short InChIKey is exactly 14 characters (first layer of the 27-character InChIKey), with no stereoisomer duplicates remaining
- Each .mgf spectrum has parent mass between 100 and 1500 Da and at most 500 peaks; entries lacking ≥6 fragments should have been excluded upstream
- The .tsv maps every entry to exactly one biological source resolved to species level against the Catalogue of Life
- Both files exist, parse cleanly (.mgf BLOCKS well-formed, .tsv tab-delimited with consistent columns), and entries are 1:1 paired across the two files

## Limitations

- 197 benchmarking entries were found to have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the problematic list is provided online as Problematic_entries.csv and erroneous entries cannot be identified by annotation tools
- All compounds in the benchmarking dataset are present in the DNP, so ISDB-DNP is structurally favored in any benchmark using this set
- GNPS spectral libraries were also part of the Sirius training set, confounding fair tool comparison on this benchmark
- Stereoisomers are not distinguished: structures are deduplicated at the first InChIKey layer, so the benchmark conflates stereoisomeric natural products

## Evidence

- [discussion] Deduplication convention: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] Taxonomy resolution filter: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] Benchmarking dataset size: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [discussion] Data availability: "Input and output data are available on OSF at the following address (https://osf.io/bvs6x/)"
- [readme] Example files note: [evidence span withheld — anchor status: not_found]
