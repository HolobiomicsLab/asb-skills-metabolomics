---
name: spectral-library-metadata-curation
description: Use when building a benchmarking or reference set of experimental MS/MS
  spectra from public spectral libraries (e.g. GNPS) and library entries need filtering,
  metadata cleaning, and structural ID standardization.
when_to_use_negative:
- When the input is already a curated, single-instrument feature table or a local
  proprietary spectral library whose entries have already been structurally resolved
- When the target analysis concerns ubiquitous (primary) metabolites rather than specialized
  metabolites, since the curated library is biased toward entries with structural
  identifiers and taxonomy-resolvable sources
- When spectra were acquired with non-protonated adducts ([M-H]-, [M+Na]+, etc.) that
  are not part of the [M+H]+ selection applied here
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: Convert retained library structures to their 27-character InChIKey used for
    deduplication by first layer
- name: mgf_filterer
  role: Script that filters .mgf spectral files according to fragment-count and peak-intensity
    parameters
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: GNPS_parser_cleaner
  role: Parses each library entry and cleans its metadata fields, keeping only entries
    with [M+H]+ adducts that possess structural identifiers
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: GNPS spectral libraries
  role: Source of the concatenated public spectral library entries subjected to filtering
    and cleaning
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:4310b4c22b6ff2fe27f45496341afb8a9d3841c1b3ff7e61b52fb1a7127e278d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/taxonomicallyinformedannotation/tima
  tools:
  - GNPS spectral libraries
  - GNPS_parser_cleaner
  - RDKit
  - mgf_filterer
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-library-metadata-curation

## Summary

Assemble a clean, structurally resolvable benchmarking set from GNPS public spectral libraries by filtering .mgf entries, cleaning metadata, converting structures to InChIKeys, and removing duplicates/stereoisomers.

## When to use

Use when building a benchmarking or reference set of experimental MS/MS spectra from public spectral libraries (e.g. GNPS) and library entries need filtering, metadata cleaning, and structural ID standardization.

## When NOT to use

- When the input is already a curated, single-instrument feature table or a local proprietary spectral library whose entries have already been structurally resolved
- When the target analysis concerns ubiquitous (primary) metabolites rather than specialized metabolites, since the curated library is biased toward entries with structural identifiers and taxonomy-resolvable sources
- When spectra were acquired with non-protonated adducts ([M-H]-, [M+Na]+, etc.) that are not part of the [M+H]+ selection applied here

## Inputs

- GNPS and third-party public spectral library .mgf files (retrieved from https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp)
- Library entry metadata fields (adduct, parent mass, structural identifiers)
- Chemical structures to be converted to InChIKey via RDKit

## Outputs

- Filtered and cleaned spectral library dataset (gnps_library_filtered.tsv) of 40,138 spectral entries
- 8,558 unique structures identified by first-layer InChIKey (first 14 characters)
- Entries restricted to [M+H]+ adducts with parent masses between 100 and 1,500 Da and spectra with ≥6 fragments / ≤500 peaks

## How to apply

Retrieve all GNPS and publicly accessible third-party spectral libraries from the GNPS libraries page (https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp) and concatenate the .mgf spectral files. Filter each library entry to retain only spectra with at least 6 fragments, and for spectra with more than 500 fragments keep only the 500 most intense peaks. Parse and clean each entry's metadata, keeping only entries with [M+H]+ protonated adducts that possess structural identifiers, and convert retained structures to their 27-character InChIKey using RDKit. Apply a parent-mass window of 100–1,500 Da and then remove duplicate structures and stereoisomers by keeping distinct InChIKey values according to the first layer (first 14 characters). Applying this pipeline to 66,646 concatenated GNPS library entries yielded a dataset of 40,138 spectral entries covering 8,558 unique first-layer InChIKeys, which serves as the source of experimental MS/MS spectra for the benchmarking set.

## Related tools

- **RDKit** (Convert retained library structures to their 27-character InChIKey used for deduplication by first layer)
- **mgf_filterer** (Script that filters .mgf spectral files according to fragment-count and peak-intensity parameters) — https://github.com/taxonomicallyinformedannotation/tima
- **GNPS_parser_cleaner** (Parses each library entry and cleans its metadata fields, keeping only entries with [M+H]+ adducts that possess structural identifiers) — https://github.com/taxonomicallyinformedannotation/tima
- **GNPS spectral libraries** (Source of the concatenated public spectral library entries subjected to filtering and cleaning)

## Examples

```
Rscript mgf_filterer.R --input gnps_library.mgf --min_fragments 6 --max_peaks 500 --output gnps_filtered.mgf
```

## Evaluation signals

- The resulting gnps_library_filtered.tsv must contain 40,138 spectral entries over 8,558 unique first-layer InChIKeys
- All retained entries have parent masses within the 100–1,500 Da window
- All retained spectra satisfy the ≥6-fragment criterion and contain no more than 500 peaks (500 most intense retained)
- All retained entries are protonated [M+H]+ adducts possessing structural identifiers resolvable to an InChIKey
- No two retained entries share the same first-layer (first 14 characters) InChIKey

## Limitations

- Only [M+H]+ protonated adducts are retained, so the curated set is not representative of other adduct types
- Duplicate structures and stereoisomers are collapsed by first-layer InChIKey, so distinct stereoisomers are removed rather than preserved
- Entries lacking structural identifiers or taxonomy-resolvable biological source information are dropped, biasing the library toward well-annotated compounds
- 197 entries of the derived benchmarking dataset had large mass differences (>0.01 Da) between the experimental parent ion mass and the calculated exact mass, indicating residual metadata errors

## Evidence

- [discussion] All GNPS libraries and publicly accessible third-party libraries were retrieved online (https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp): "All GNPS libraries and publicly accessible third-party libraries were retrieved online (https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp)"
- [discussion] Filter .mgf files to retain only entries with ≥6 fragments; keep 500 most intense peaks otherwise: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] Keep entries with parent masses between 100 and 1,500 Da: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] Convert structures to InChIKey using RDKit: "converting structures to their InChIKey, a 27-character hashed version of the full InChI"
- [discussion] Remove duplicates and stereoisomers by first 14 InChIKey characters: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [results] Resulting dataset of 40,138 structures (8,558 unique structures) after preprocessing 66,646 GNPS entries: [evidence span withheld — anchor status: not_found]
- [discussion] Scripts created to filter .mgf spectral files according to specific parameters: "created to filter.mgf spectral file according to specific parameters"
- [discussion] 197 benchmarking entries had large mass difference (>0.01 Da) between experimental parent ion mass and calculated exact mass: "a number of entries (197) of the benchmarking dataset were found to have large mass difference (> 0.01 Da)"
