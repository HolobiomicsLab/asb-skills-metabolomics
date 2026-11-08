---
name: interactive-datatable-construction
description: Use when publishing curated chemical or spectral tables (e.g., fragment
  ions with m/z, intensity, precursor, collision energy) as an interactive, searchable
  HTML datatable for public repositories.
when_to_use_negative:
- The goal is statistical analysis or data manipulation of the table rather than publication-grade
  interactive display — use data.table or Tidyverse alone instead of DT
- The fragment data have not yet been filtered (e.g., CFM-ID fragments below the 5%
  relative intensity threshold still included), since the datatables were built from
  thresholded, deduplicated entries
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DT
  role: R package used to render the filtered fragment tables as searchable interactive
    web datatables
  repo: https://github.com/rstudio/DT
- name: R
  role: R Project for Statistical Computing (version 4.1.1), the environment in which
    the datatables are assembled with DT and Tidyverse
- name: Tidyverse
  role: R packages used alongside DT to assemble and shape the fragment datatable
    rows
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:1527d0f4ce2532696fc31ecf174c92fb9daef7e4d8781fc5b79f924c981b7891
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - DT
  - R
  - Tidyverse
  - tandem HRMS/MS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# interactive-datatable-construction

## Summary

Assembles curated fragment-ion tables (in silico CFM-ID predictions for 582 DNA adducts and experimental MS/MS fragments from 15 reference standards) into searchable, interactive web datatables using R's DT package with Tidyverse, as deployed for the DNA adductomics database.

## When to use

Use when publishing curated chemical or spectral tables (e.g., fragment ions with m/z, intensity, precursor, collision energy) as an interactive, searchable HTML datatable for public repositories.

## When NOT to use

- The goal is statistical analysis or data manipulation of the table rather than publication-grade interactive display — use data.table or Tidyverse alone instead of DT
- The fragment data have not yet been filtered (e.g., CFM-ID fragments below the 5% relative intensity threshold still included), since the datatables were built from thresholded, deduplicated entries

## Inputs

- CFM-ID in silico MS/MS fragment output for 582 DNA adduct structures at three energy levels (filtered to >5% relative intensity)
- Experimental MS/MS fragment entries for 15 DNA adduct reference standards at collision energies 20, 40, and 60 eV (from UNIFI-exported .xlsx spectra files)
- Parent adduct annotations (name, precursor ion, SMILES/InChI/InChIKey) from the DNA adduct database .xlsx file

## Outputs

- Searchable interactive web datatable of predicted fragments ('The database of predicted fragments') with one row per fragment ion annotated by parent adduct
- Searchable interactive web datatable of experimental fragments ('the database of experimental fragments') with standard name, precursor ion, fragment m/z, and collision energy per row

## How to apply

First prepare the underlying data: filter CFM-ID in silico MS/MS output to retain only fragment ions above 5% relative intensity (one entry per fragment ion), or organize experimental MS/MS fragments from the 15 reference standards by the three collision energies (20, 40, 60 eV). Load the filtered rows in R (version 4.1.1) using the DT package together with Tidyverse, annotating each predicted fragment with its parent adduct and including fields such as standard name, precursor ion, fragment m/z, and collision energy. Render the table with DT::datatable() so users can search and sort interactively, then deposit the resulting datatable in the public GitLab repository alongside the .xlsx, .cdxml, and .mol files for the 582 adduct entries. The rationale is that web datatables make both experimental and predicted fragments searchable without requiring users to parse raw spectral exports (e.g., UNIFI-exported .xlsx files).

## Related tools

- **DT** (R package used to render the filtered fragment tables as searchable interactive web datatables) — https://github.com/rstudio/DT
- **R** (R Project for Statistical Computing (version 4.1.1), the environment in which the datatables are assembled with DT and Tidyverse)
- **Tidyverse** (R packages used alongside DT to assemble and shape the fragment datatable rows)

## Examples

```
install.packages('DT'); DT::datatable(fragments_df)
```

## Evaluation signals

- The predicted-fragments datatable contains fragment entries for all 582 adducts at three energy levels, with only fragments above 5% relative intensity retained
- The experimental-fragments datatable covers all 15 reference standards (e.g., N6-Me-dA, 8-oxo-dG, M1-dG) across collision energies 20, 40, and 60 eV
- Each predicted fragment row is annotated with its parent adduct, and each experimental row includes standard name, precursor ion, fragment m/z, and collision energy
- The datatable reproduces the deposited tables in https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database when diffed against the repository copy
- One row is kept per fragment ion (no duplicate fragment entries) to preserve searchability

## Limitations

- The DT package is in maintenance-only mode: priority is fixing important bugs, and pull requests cannot be guaranteed review or merge
- Only fragments above 5% relative intensity are included, so low-intensity predicted fragments are absent from the searchable datatable
- The experimental datatable is limited to the 15 purchased reference standards; adducts without standards have only CFM-ID predicted fragments

## Evidence

- [abstract] filtering to predicted fragments above 5% intensity: [evidence span withheld — anchor status: not_found]
- [abstract] web datatables of experimental and predicted fragments: "reporting both experimental and predicted fragments into interactive web datatables"
- [readme] R DT datatable rendering: [evidence span withheld — anchor status: external_doc]
- [readme] DT maintenance mode caveat: [evidence span withheld — anchor status: external_doc]
