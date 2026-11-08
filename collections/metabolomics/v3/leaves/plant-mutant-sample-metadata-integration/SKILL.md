---
name: plant-mutant-sample-metadata-integration
description: Use when raw metabolomics or expression data tables and their sample
  metadata (e.g., mutation genotype/knockout conditions retrieved from PMR) exist
  as separate files and must be combined into one integrated, metadata-linked MOG
  project.
when_to_use_negative:
- Input is already a saved MOG project (.mog) with metadata linked — load it directly
  in MOG rather than rebuilding
- Data lacks any accompanying sample metadata (e.g., genotype/condition descriptors),
  so no metadata-to-column linkage can be formed
- Analysis requires raw chromatogram processing (e.g., converting GC-MS instrument
  files to metabolite tables) rather than integrating already-tabulated metabolite
  accumulation values
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0219
tools:
- name: MetaOmGraph (MOG)
  role: Java workbench used to import the metabolite data table and metadata table,
    link sample metadata to data columns, and save the integrated project as AT-Mutation-242-656-metab.mog
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MOG
  role: Executable name under which MetaOmGraph is downloaded (zip containing a .jar
    started by double-click) to build and save the annotated project
  repo: https://github.com/urmi-21/MetaOmGraph/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:ecf5b3f20417d70888b8df1811a3a56383e9622950aaafa55c711f271ff899db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - MetaOmGraph (MOG)
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# plant-mutant-sample-metadata-integration

## Summary

Constructs a metadata-annotated MetaOmGraph (MOG) project, such as AT-Mutation-242-656-metab.mog, by linking a GC-MS metabolite accumulation table for 656 Arabidopsis thaliana samples (50 knockout/knockdown mutants from PMR) to its sample metadata, producing a savable, shareable .mog project file for interactive exploratory analysis.

## When to use

Use when raw metabolomics or expression data tables and their sample metadata (e.g., mutation genotype/knockout conditions retrieved from PMR) exist as separate files and must be combined into one integrated, metadata-linked MOG project.

## When NOT to use

- Input is already a saved MOG project (.mog) with metadata linked — load it directly in MOG rather than rebuilding
- Data lacks any accompanying sample metadata (e.g., genotype/condition descriptors), so no metadata-to-column linkage can be formed
- Analysis requires raw chromatogram processing (e.g., converting GC-MS instrument files to metabolite tables) rather than integrating already-tabulated metabolite accumulation values

## Inputs

- GC-MS metabolite accumulation data table for 656 samples / 50 A. thaliana mutants (from the AT-metab-dataset, retrieved from PMR)
- Sample metadata table describing mutation conditions (knockout/knockdown gene targets) for each sample (from PMR)

## Outputs

- MetaOmGraph project file AT-Mutation-242-656-metab.mog containing the integrated metabolomics dataset with sample metadata linked to data columns
- Verified dataset spanning 656 samples across 50 mutations, queryable by metadata attributes within MOG

## How to apply

Download the GC-MS metabolite accumulation data and the accompanying sample metadata (mutation knockout/knockdown conditions) for the 50 A. thaliana mutants from the PMR resource. Import the metabolite data table and metadata table into MetaOmGraph (a Java workbench requiring Java Runtime Environment 8+, with optional R 3.4+ for extra analyses), and link each metadata entry to its corresponding data column so every one of the 656 samples carries its mutation annotation. MOG incorporates metadata to 'add another dimension to the analyses', enabling interactive filtering and visualization in the context of sample attributes. Verify the assembled project spans all 656 samples across the 50 mutations, then save the integrated dataset as a MOG project file (AT-Mutation-242-656-metab.mog); MOG projects preserve the history of explorations and can be shared, reused, and included in publications.

## Related tools

- **MetaOmGraph (MOG)** (Java workbench used to import the metabolite data table and metadata table, link sample metadata to data columns, and save the integrated project as AT-Mutation-242-656-metab.mog) — https://github.com/urmi-21/MetaOmGraph/
- **MOG** (Executable name under which MetaOmGraph is downloaded (zip containing a .jar started by double-click) to build and save the annotated project) — https://github.com/urmi-21/MetaOmGraph/

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- Assembled project contains exactly 656 samples covering all 50 knockout/knockdown mutations
- Every data column in the .mog project is linked to a metadata entry describing its mutation condition
- Opening AT-Mutation-242-656-metab.mog in MOG allows metadata-driven filtering and visualization (e.g., grouping samples by mutation) without further linkage steps
- Saved project file loads cleanly in a fresh MOG installation (Java 8+) and preserves the exploration history
- Metabolite values in the project match the source PMR AT-metab-dataset table (no rows or samples lost during import)

## Limitations

- Requires Java Runtime Environment 8 or higher; R 3.4+ is optional and only needed for sending data subsets to R
- MOG projects with data indexing/buffering are designed for large expression-style data; data must be tabulated with a mappable sample identifier to metadata, not raw instrument output
- Metadata integration is only as correct as the PMR metadata — sample-to-column linkage errors propagate into all downstream MOG analyses

## Evidence

- [results] Source data and workflow: "Here, we use the AT-metab-dataset metabolomics dataset that represents a comprehensive study of 50 mutants"
- [results] Metadata provenance: "Data and metadata were retrieved from PMR (61)."
- [readme] MOG metadata integration rationale: "By incorporating metadata, MOG adds another dimension to the analyses and provides flexibility in data exploration."
- [readme] Savable/shareable projects: [evidence span withheld — anchor status: external_doc]
- [readme] Prerequisites: [evidence span withheld — anchor status: external_doc]
