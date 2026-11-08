---
name: gc-ms-metabolite-data-retrieval
description: Use when starting from a public metabolomics resource entry (e.g., PMR)
  with GC-MS metabolite accumulation tables plus mutation/condition metadata, and
  the goal is a metadata-linked MOG project for interactive exploratory analysis.
when_to_use_negative:
- The data are already loaded into a MOG project with metadata linked — use MOG's
  exploratory analysis features (correlation, differential expression) instead of
  re-importing.
- The source is raw GC-MS instrument output (e.g., NetCDF/mzML chromatograms) rather
  than a processed metabolite accumulation table from PMR — this skill assumes curated
  accumulation data.
- The metadata cannot be linked unambiguously to data columns (e.g., no shared sample
  identifiers), since the value of the project depends on metadata integration.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MetaOmGraph
  role: Imports the metabolite data table and metadata, links metadata to data columns,
    and saves the integrated dataset as a savable/shareable MOG project file (.mog)
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: PMR
  role: Public metabolomics resource from which the GC-MS metabolite accumulation
    data and mutation-condition metadata for the 50 mutants are retrieved
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:712bf9e42f47f36bdeabb3ca32545718d1312644e4f1fec16f2a6866756f714b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - PMR
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gc-ms-metabolite-data-retrieval

## Summary

Retrieves GC-MS metabolite accumulation data and accompanying sample metadata from the PMR resource and assembles them into an integrated, metadata-annotated MetaOmGraph (MOG) project, as done for the 656-sample Arabidopsis AT-metab-dataset covering 50 knockout/knockdown mutants (AT-Mutation-242-656-metab.mog).

## When to use

Use when starting from a public metabolomics resource entry (e.g., PMR) with GC-MS metabolite accumulation tables plus mutation/condition metadata, and the goal is a metadata-linked MOG project for interactive exploratory analysis.

## When NOT to use

- The data are already loaded into a MOG project with metadata linked — use MOG's exploratory analysis features (correlation, differential expression) instead of re-importing.
- The source is raw GC-MS instrument output (e.g., NetCDF/mzML chromatograms) rather than a processed metabolite accumulation table from PMR — this skill assumes curated accumulation data.
- The metadata cannot be linked unambiguously to data columns (e.g., no shared sample identifiers), since the value of the project depends on metadata integration.

## Inputs

- GC-MS metabolite accumulation data table for 50 A. thaliana mutants (AT-metab-dataset) from PMR
- Sample metadata table describing knockout/knockdown mutation conditions from PMR

## Outputs

- Integrated, metadata-annotated MetaOmGraph project file AT-Mutation-242-656-metab.mog containing 656 GC-MS samples across 50 mutations

## How to apply

Download the GC-MS metabolite accumulation data table for the target study from PMR — in this case the AT-metab-dataset covering 50 A. thaliana knockout/knockdown mutants. Retrieve the accompanying sample metadata describing each mutation condition from PMR ('Data and metadata were retrieved from PMR'). Import both the metabolite data table and the metadata table into MetaOmGraph, then link the sample metadata to the data columns so each of the 656 GC-MS samples is annotated with its mutation. Verify the assembled project contains 656 samples spanning the 50 mutations, then save it as a MOG project file named AT-Mutation-242-656-metab.mog, which preserves the metadata linkage for sharing and reuse.

## Related tools

- **MetaOmGraph** (Imports the metabolite data table and metadata, links metadata to data columns, and saves the integrated dataset as a savable/shareable MOG project file (.mog)) — https://github.com/urmi-21/MetaOmGraph/
- **PMR** (Public metabolomics resource from which the GC-MS metabolite accumulation data and mutation-condition metadata for the 50 mutants are retrieved)

## Evaluation signals

- The saved MOG project contains exactly 656 samples covering the 50 knockout/knockdown mutations of A. thaliana genes.
- Every data column in the project is annotated with its corresponding sample metadata (mutation condition) — no unlinked columns remain.
- The project opens in MetaOmGraph as a valid .mog file and its metadata can be used to filter/sort samples (e.g., by mutation).
- Metabolite values in the project match the source PMR AT-metab-dataset accumulation table (no accidental reordering of columns during metadata linkage).

## Limitations

- The skill depends on PMR providing both the accumulation data and consistent sample metadata; ambiguous or missing sample identifiers would break the metadata-to-column linkage.
- MOG is a Java desktop application (requires Java Runtime Environment 8+) and, optionally, R 3.4+ for sending subsets to R; it processes local files rather than querying remote resources directly.
- The curated dataset is study-specific (656 GC-MS samples, 50 mutants); reusing the skill for other studies requires equivalent data and metadata tables from the relevant resource.

## Evidence

- [results] PMR retrieval of metabolomics data and metadata: "Data and metadata were retrieved from PMR (61)."
- [results] AT-metab-dataset scope: "Here, we use the AT-metab-dataset metabolomics dataset that represents a comprehensive study of 50 mutants"
- [abstract] MOG project saving and sharing: "MOG projects, with history of explorations, can be saved and shared."
- [readme] Metadata integration in MOG projects: [evidence span withheld — anchor status: external_doc]
