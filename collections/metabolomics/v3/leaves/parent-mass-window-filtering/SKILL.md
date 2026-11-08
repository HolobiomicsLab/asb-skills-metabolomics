---
name: parent-mass-window-filtering
description: Use when assembling or curating MS/MS spectral libraries from .mgf entries
  and you need to discard ions outside the mass range typical of natural-product metabolites
  before dereplication or benchmarking.
when_to_use_negative:
- When the input is already a feature table or peak list from a profiling experiment
  rather than a spectral library in .mgf format — the filter is a library-curation
  step, not a feature-detection step
- When annotating primary or ubiquitous metabolites outside the 100–1,500 Da range,
  since the window is tuned to specialized natural-product metabolites in the GNPS
  libraries
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: mgf_filterer
  role: Applies the .mgf filtering chain — including the 100–1,500 Da parent-mass
    window — to concatenated GNPS spectral library files
- name: GNPS_parser_cleaner
  role: Parses and cleans library metadata (keeping [M+H]+ adducts with structural
    identifiers) before the parent-mass filter is applied
- name: RDKit
  role: Converts the retained structures to InChIKey so duplicate structures and stereoisomers
    can be removed after mass filtering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:e4dd95ae54d625513beb8e0a176ce977c5cadbe6bd0d928e4539bfdf5fa9df5b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS_parser_cleaner
  - RDKit
  - mgf_filterer
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# parent-mass-window-filtering

## Summary

A preprocessing filter that restricts spectral library entries to those whose parent ion masses fall within a biologically relevant window (100–1,500 Da), used when assembling the GNPS-derived benchmarking dataset for MS/MS annotation tool evaluation.

## When to use

Use when assembling or curating MS/MS spectral libraries from .mgf entries and you need to discard ions outside the mass range typical of natural-product metabolites before dereplication or benchmarking.

## When NOT to use

- When the input is already a feature table or peak list from a profiling experiment rather than a spectral library in .mgf format — the filter is a library-curation step, not a feature-detection step
- When annotating primary or ubiquitous metabolites outside the 100–1,500 Da range, since the window is tuned to specialized natural-product metabolites in the GNPS libraries

## Inputs

- Concatenated GNPS spectral library .mgf files (66,646 entries) with per-entry parent mass metadata
- Parsed and cleaned library entries restricted to [M+H]+ adducts with structural identifiers

## Outputs

- Filtered .mgf or TSV table retaining only entries with parent masses between 100 and 1,500 Da (40,138 entries in the article's pipeline)
- Entry-level parent mass column compliant with the 100–1,500 Da window for downstream InChIKey deduplication

## How to apply

Retrieve GNPS and third-party spectral libraries as .mgf files, parse each library entry, and clean metadata keeping only [M+H]+ protonated adducts that possess structural identifiers. Apply the mass filter by keeping only entries whose parent masses are comprised between 100 and 1,500 Da. In the Rutz et al. pipeline this filter is applied alongside a minimum-fragment filter (retain entries with at least 6 fragments, keeping only the 500 most intense peaks when more fragments are present) and a structural-deduplication step (retain distinct InChIKey first layers, i.e. the first 14 characters) — the mass window is one stage in a sequential .mgf filtering chain that ultimately produces 40,138 entries over 8,558 unique structures.

## Related tools

- **mgf_filterer** (Applies the .mgf filtering chain — including the 100–1,500 Da parent-mass window — to concatenated GNPS spectral library files)
- **GNPS_parser_cleaner** (Parses and cleans library metadata (keeping [M+H]+ adducts with structural identifiers) before the parent-mass filter is applied)
- **RDKit** (Converts the retained structures to InChIKey so duplicate structures and stereoisomers can be removed after mass filtering)

## Examples

```
Rscript mgf_filterer.R --input gnps_libraries.mgf --min-fragments 6 --max-peaks 500 --parent-min 100 --parent-max 1500 --output gnps_library_filtered.tsv
```

## Evaluation signals

- The resulting gnps_library_filtered.tsv must contain 40,138 spectral entries over 8,558 unique first-layer InChIKeys
- All retained entries must have parent masses within the 100–1,500 Da window
- All retained spectra must satisfy the ≥6-fragment and 500-peak criteria alongside the mass window
- Only [M+H]+ protonated adducts with structural identifiers should remain after parsing/cleaning and before mass filtering

## Limitations

- 197 benchmarking entries were found to have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the parent-mass filter does not catch these inconsistencies
- The 100–1,500 Da window is dataset-specific (GNPS library entries for natural products) and may discard legitimate ions outside that range in other contexts
- The filter depends on correctly reported parent masses in library metadata; erroneous or rounded masses propagate through

## Evidence

- [methods] Retain entries with parent masses between 100 and 1,500 Da: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [methods] Filtering of .mgf spectral files: "created to filter.mgf spectral file according to specific parameters"
- [discussion] Minimum fragment count and peak-intensity cap: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] Dataset size after all filters: [evidence span withheld — anchor status: not_found]
- [discussion] Deduplication by InChIKey first layer: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] GNPS libraries retrieval: "All GNPS libraries and publicly accessible third-party libraries were retrieved online (https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp)"
- [discussion] Parent mass vs exact mass inconsistency: "a number of entries (197) of the benchmarking dataset were found to have large mass difference (> 0.01 Da)"
