---
name: metabolomics-workbench-data-retrieval
description: Use when a pipeline must fetch a metabolomics dataset (metabolite names/concentrations
  plus metadata) for a specific Metabolomics Workbench study ID (e.g. ST000915, ST001308,
  ST001140) as the entry point of an enrichment workflow.
when_to_use_negative:
- The dataset is a local/custom file rather than a deposited Metabolomics Workbench
  study — use significant_met_own with your own metabolite and metadata tables instead
  of getmwstudies
- Data are already loaded as a metabolite-by-sample intensity data frame with matching
  metadata; retrieval would be redundant
- The study of interest has no metabolite-name or concentration data compatible with
  RefMet standardization (names are unstructured without usable identifiers)
edam_operation: http://edamontology.org/operation_2422
tools:
- name: MetENP
  role: Provides getmwstudies(studyid, typedata) to fetch metabolite and metadata
    tables for a study directly from Metabolomics Workbench
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: Metabolomics Workbench
  role: Data repository (NMDR) hosting the metabolomics datasets that getmwstudies
    queries; also hosts the MetENPweb interface over the same studies
- name: RefMet
  role: Standard nomenclature applied to retrieved metabolite names via convert_refmet
    so downstream KEGG mapping succeeds
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_002
  - task_004
  - task_001
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:c172a3b75b3fab9330e33429b676500df03ef4046121401743108b72c6a2e480
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - ADViSELipidomics
  - KEGGREST
  - MetENP
  - Metabolomics Workbench
  - R
  - RefMet
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolomics-workbench-data-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves metabolite abundance tables and associated sample metadata from Metabolomics Workbench (NMDR) studies via MetENP's getmwstudies(studyid, typedata) function, providing the structured input tables that every downstream MetENP enrichment step (significant_met, convert_refmet, map_keggid) consumes.

## When to use

Use when a pipeline must fetch a metabolomics dataset (metabolite names/concentrations plus metadata) for a specific Metabolomics Workbench study ID (e.g. ST000915, ST001308, ST001140) as the entry point of an enrichment workflow.

## When NOT to use

- The dataset is a local/custom file rather than a deposited Metabolomics Workbench study — use significant_met_own with your own metabolite and metadata tables instead of getmwstudies
- Data are already loaded as a metabolite-by-sample intensity data frame with matching metadata; retrieval would be redundant
- The study of interest has no metabolite-name or concentration data compatible with RefMet standardization (names are unstructured without usable identifiers)

## Inputs

- Metabolomics Workbench study identifier (e.g. ST000915, ST001308, ST001140)
- typedata selector specifying whether to fetch the metabolite abundance table ('data') or the sample metadata table
- List of analytical method types to iterate over (for multi-method studies such as ST001140)

## Outputs

- Metabolite data frame of metabolite names (or names plus concentrations) per sample, one row per metabolite
- Metadata data frame mapping sample IDs to experimental factor levels (e.g. Normal vs Cirrhosis, ridA-mutant vs wild type, prednisolone vs tetracosactide)
- CSV exports of the retrieved tables for downstream significant_met / convert_refmet steps

## How to apply

Call MetENP's getmwstudies(studyid, typedata) with the MW study identifier and the desired data type to obtain the metabolite-intensity table and the metadata table; the workflow cards retrieve data across all analytical method types for multi-method studies (e.g. ST001140 canine plasma lipidomics, four method types) or a single method table (e.g. ST000915, ST001308). Verify the returned data frame carries metabolite names plus concentrations and a metadata frame with sample IDs and the factor columns (e.g. 'Genotype' for ST001308, treatment factor for ST001140) needed by significant_met's factor1/factor2/factor_col/sample_col arguments. Standardize retrieved metabolite names with convert_refmet before mapping to KEGG compound IDs, since downstream species-specific pathway mapping (sps='hsa', Salmonella, dog 'dga') depends on the RefMet standard nomenclature. This retrieval step is the first stage of the MetENP pipeline, which was designed so analysis happens directly 'within the Metabolomics Workbench' on deposited studies.

## Related tools

- **MetENP** (Provides getmwstudies(studyid, typedata) to fetch metabolite and metadata tables for a study directly from Metabolomics Workbench) — https://github.com/metabolomicsworkbench/MetENP
- **Metabolomics Workbench** (Data repository (NMDR) hosting the metabolomics datasets that getmwstudies queries; also hosts the MetENPweb interface over the same studies)
- **RefMet** (Standard nomenclature applied to retrieved metabolite names via convert_refmet so downstream KEGG mapping succeeds)

## Examples

```
library(MetENP); metdata <- getmwstudies("ST000915", "data")
```

## Evaluation signals

- The retrieved metabolite table has one row per metabolite and columns for every sample in the metadata table, with sample IDs joinable via the sample_col argument
- Metadata table contains the factor column and exactly the two comparison groups later passed as factor1/factor2 to significant_met (e.g. 'Genotype' with ridA-mutant and wild type)
- Retrieval repeated across all analytical method types of a multi-method study (ST001140) yields the union that produces the reported 26 significant metabolites and 7 KEGG pathways at p=0.05, log2FC 0.5, 50percent NA filter
- Metabolite names in the retrieved table convert through convert_refmet with non-missing main_class, sub_class, and super_class, confirming the names are mappable
- Study identifier resolves on Metabolomics Workbench (prefix ST) and the fetch runs without error in R after KEGGREST and other Bioconductor dependencies are installed

## Limitations

- MetENP provides no explicit installation via CRAN/Bioconductor — only GitHub source (git clone) or Anaconda environment.yml — so retrieval requires installing KEGGREST, KEGGgraph, and pathview via BiocManager first
- No software version, changelog, or license is stated in the article, so reproducibility of retrieved data depends on the current GitHub version
- Kegg-based retrieval limits downstream pathway analysis to species with available KEGG pathways; studies without such species annotation cannot be fully mapped after retrieval

## Evidence
