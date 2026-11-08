---
name: fragment-intensity-threshold-filtering
description: Use when in silico MS/MS fragment predictions have been generated (e.g.,
  by CFM-ID) at multiple energy levels and need to be reduced to the most informative
  ions before publication or spectral-library search.
when_to_use_negative:
- When processing experimental MS/MS spectra of the 15 reference standards — those
  are reported separately in the experimental spectral-library datatable, not intensity-thresholded
  from CFM-ID output
- When the input is already a curated fragment table or an interactive web datatable
  rather than raw CFM-ID prediction output
- When absolute (not relative) fragment intensities are the target quantity, since
  the 5% threshold is defined on relative intensity
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CFM-ID
  role: Generates the in silico MS/MS fragments for all 582 adducts at three energy
    levels that are subsequently intensity-filtered
  repo: https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- name: DT (R package)
  role: Renders the filtered, >5% intensity fragment entries into the searchable interactive
    web datatable 'The database of predicted fragments'
  repo: https://github.com/rstudio/DT
- name: Tidyverse
  role: Used with DT in R 4.1.1 to assemble and annotate the filtered fragment datatable
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:f9339f421fd415814e9cb91bd5edf32b0a7b56502750c6d1f2454c63a82ca4e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - DT
  - DT (R package)
  - R
  - Tidyverse
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fragment-intensity-threshold-filtering

## Summary

Filters in silico MS/MS fragment predictions (e.g., CFM-ID output for the 582 DNA adducts) by relative intensity, retaining only fragment ions above a 5% threshold to keep the searchable datatable concise and searchable.

## When to use

Use when in silico MS/MS fragment predictions have been generated (e.g., by CFM-ID) at multiple energy levels and need to be reduced to the most informative ions before publication or spectral-library search.

## When NOT to use

- When processing experimental MS/MS spectra of the 15 reference standards — those are reported separately in the experimental spectral-library datatable, not intensity-thresholded from CFM-ID output
- When the input is already a curated fragment table or an interactive web datatable rather than raw CFM-ID prediction output
- When absolute (not relative) fragment intensities are the target quantity, since the 5% threshold is defined on relative intensity

## Inputs

- CFM-ID in silico MS/MS fragment predictions for all 582 DNA adducts at three energy levels
- Structure identifiers (.mol files, SMILES, InChI, InChIKey) from the database .xlsx file
- Relative intensity values for each predicted fragment ion

## Outputs

- Filtered datatable of fragment ions with relative intensity > 5%, one entry per fragment ion, annotated with parent adduct
- Interactive web datatable ('The database of predicted fragments') rendered with the R DT package

## How to apply

Run CFM-ID (wishartlab/cfmid docker image) on each adduct structure (.mol/SMILES/InChI) to generate in silico MS/MS fragments at three energy levels, matching the three-level scheme used for experimental spectra. Filter the CFM-ID output to retain only fragment ions whose relative intensity exceeds the 5% threshold, keeping one entry per fragment ion to improve searchability. Assemble the retained fragments in R (version 4.1.1) using the DT and Tidyverse packages into a web datatable, annotating each fragment with its parent adduct. The rationale is that low-intensity predicted fragments add noise without aiding annotation in untargeted HRMS DNA adductomics.

## Related tools

- **CFM-ID** (Generates the in silico MS/MS fragments for all 582 adducts at three energy levels that are subsequently intensity-filtered) — https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- **DT (R package)** (Renders the filtered, >5% intensity fragment entries into the searchable interactive web datatable 'The database of predicted fragments') — https://github.com/rstudio/DT
- **Tidyverse** (Used with DT in R 4.1.1 to assemble and annotate the filtered fragment datatable)

## Examples

```
library(DT); fragments_5pct <- subset(cfmid_fragments, rel_intensity > 5); datatable(fragments_5pct)
```

## Evaluation signals

- The datatable contains fragment entries for all 582 adducts at three energy levels
- Every retained fragment has relative intensity strictly above the 5% threshold; no fragments at or below 5% remain
- One entry per fragment ion is kept (no duplicate fragment ions for the same parent adduct and energy level)
- Each fragment entry is annotated with its parent adduct and appears in the interactive web datatable consistent with the paper's reported predicted-fragment datatable

## Limitations

- The 5% cutoff is applied to CFM-ID relative intensities, so genuinely informative low-intensity fragments are excluded from the searchable datatable
- The DT package is in maintenance-only mode, so new feature contributions may not be merged
- Filtering is applied per fragment ion at the prediction level; it does not validate predictions against the experimental MS/MS spectra of the 15 reference standards

## Evidence

- [abstract] generating in silico MS/MS fragments for all the adducts: "a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards, generating in silico MS/MS fragments"
- [abstract] reporting both experimental and predicted fragments into interactive web datatables: "reporting both experimental and predicted fragments into interactive web datatables"
- [readme] This package provides a function datatable() to display R data: [evidence span withheld — anchor status: external_doc]
