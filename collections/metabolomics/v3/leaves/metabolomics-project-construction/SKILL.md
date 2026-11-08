---
name: metabolomics-project-construction
description: Use when raw metabolite accumulation tables (e.g. GC-MS data for mutants)
  and their condition metadata must be imported into MetaOmGraph, linked, and saved
  as a shareable .mog project.
when_to_use_negative:
- The data are RNA-seq or microarray expression data rather than metabolite accumulation
  tables — use MOG's expression-data workflows instead of this metabolomics project
  construction.
- The user only needs to analyze an existing .mog project interactively (charts, box
  plots, volcano plots); no project construction is required.
- The metadata cannot be linked to data columns (e.g. mismatched sample identifiers),
  since MOG's metadata-contextualized analyses depend on that linkage.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetaOmGraph (MOG)
  role: Java workbench used to import the metabolite data table, integrate the sample
    metadata, verify the 656-sample/50-mutation dataset, and save it as a .mog project
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: PMR
  role: Source resource from which the AT-metab-dataset metabolite data and mutation
    metadata were retrieved
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:4901d832df01321a65fe0ddba62235fad1dc20c1fab1527b43439b3c5a1355c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  - MetaOmGraph (MOG)
  - PMR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metabolomics-project-construction

## Summary

Assembles a metabolomics data matrix plus its sample metadata into an integrated, savable MetaOmGraph (MOG) project file — e.g. building AT-Mutation-242-656-metab.mog from 656 GC-MS samples spanning 50 A. thaliana knockout/knockdown mutants retrieved from PMR.

## When to use

Use when raw metabolite accumulation tables (e.g. GC-MS data for mutants) and their condition metadata must be imported into MetaOmGraph, linked, and saved as a shareable .mog project.

## When NOT to use

- The data are RNA-seq or microarray expression data rather than metabolite accumulation tables — use MOG's expression-data workflows instead of this metabolomics project construction.
- The user only needs to analyze an existing .mog project interactively (charts, box plots, volcano plots); no project construction is required.
- The metadata cannot be linked to data columns (e.g. mismatched sample identifiers), since MOG's metadata-contextualized analyses depend on that linkage.

## Inputs

- GC-MS metabolite accumulation data table for 50 A. thaliana knockout/knockdown mutants (AT-metab-dataset from PMR)
- Sample metadata table describing the mutation (knockout/knockdown) conditions, retrieved from PMR

## Outputs

- MetaOmGraph project file AT-Mutation-242-656-metab.mog integrating 656 GC-MS samples with linked mutation metadata

## How to apply

Download the GC-MS metabolite accumulation data (the AT-metab-dataset covering 50 mutants of A. thaliana genes) and the accompanying sample metadata describing knockout/knockdown conditions from the PMR resource. In MetaOmGraph (MOG), import the metabolite data table, then import the metadata table and link each metadata entry to the corresponding data columns so samples can later be grouped by mutation. Verify the integrated project contains all expected samples — 656 GC-MS samples across the 50 mutations — before saving. Save the metadata-annotated dataset as a MOG project file (AT-Mutation-242-656-metab.mog); MOG projects preserve the history of explorations and can be shared, reused, and included in publications.

## Related tools

- **MetaOmGraph (MOG)** (Java workbench used to import the metabolite data table, integrate the sample metadata, verify the 656-sample/50-mutation dataset, and save it as a .mog project) — https://github.com/urmi-21/MetaOmGraph/
- **PMR** (Source resource from which the AT-metab-dataset metabolite data and mutation metadata were retrieved)

## Evaluation signals

- The saved .mog project contains exactly 656 samples spanning the 50 knockout/knockdown mutations.
- Every data column in the MOG project is linked to a metadata entry describing its mutation condition (no unannotated samples).
- The project file opens in MetaOmGraph and supports metadata-driven filtering/grouping of samples by mutation.
- Sample counts per mutation match the source PMR metadata, and no samples were dropped during import.

## Limitations

- MOG requires a Java Runtime Environment 8 (or higher); R 3.4+ is optional and only needed for sending data subsets to R.
- The .mog project is only as good as the metadata linkage — mislinked columns would corrupt downstream mutation-grouped analyses.
- MOG is designed for all types of data but specialized for expression data, so non-expression-specific operations may be limited for metabolomics use.

## Evidence

- [results] AT-metab-dataset scope: "Here, we use the AT-metab-dataset metabolomics dataset that represents a comprehensive study of 50 mutants"
- [results] Data source: "Data and metadata were retrieved from PMR (61)."
- [readme] MOG project saving: [evidence span withheld — anchor status: external_doc]
- [readme] Metadata integration purpose: "By incorporating metadata, MOG adds another dimension to the analyses and provides flexibility in data exploration."
