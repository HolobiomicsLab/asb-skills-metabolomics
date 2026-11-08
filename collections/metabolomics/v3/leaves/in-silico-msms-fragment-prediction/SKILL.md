---
name: in-silico-msms-fragment-prediction
description: Use when building a spectral library for adductomics or metabolomics
  where reference standards are unavailable and in silico MS/MS fragments are needed
  for each structure entry.
when_to_use_negative:
- When experimental MS/MS spectra of reference standards are already available for
  the target analytes, since measured spectra take precedence over in silico predictions.
- When the goal is de novo structure elucidation rather than populating a searchable
  spectral-library datatable for known or hypothesized adduct entries.
- When the adduct structures lack computer-readable identifiers (SMILES, InChI, InChIKey,
  or .mol files) that CFM-ID requires.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID
  role: Used as a docker image (wishartlab/cfmid) to generate in silico MS/MS fragments
    for all 582 DNA adduct structures at three energy levels.
- name: R (version 4.1.1)
  role: Used with Tidyverse packages to assemble and filter the predicted fragment
    datatable.
- name: DT
  role: R package used to render the predicted fragment datatable as an interactive
    web table for searchability.
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:8b4160572969988a10def028628141997da2767300c217843339313dc3eb023f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - DT
  - R
  - R (version 4.1.1)
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# in-silico-msms-fragment-prediction

## Summary

Generates predicted MS/MS fragment ions for chemical structures (e.g., DNA adducts) using CFM-ID across three energy levels, then filters and assembles the fragments into a searchable datatable alongside the parent adduct entries.

## When to use

Use when building a spectral library for adductomics or metabolomics where reference standards are unavailable and in silico MS/MS fragments are needed for each structure entry.

## When NOT to use

- When experimental MS/MS spectra of reference standards are already available for the target analytes, since measured spectra take precedence over in silico predictions.
- When the goal is de novo structure elucidation rather than populating a searchable spectral-library datatable for known or hypothesized adduct entries.
- When the adduct structures lack computer-readable identifiers (SMILES, InChI, InChIKey, or .mol files) that CFM-ID requires.

## Inputs

- Curated database spreadsheet (.xlsx) containing structure identifiers (SMILES, InChI, InChIKey) for each adduct entry
- Structure files (.mol, .cdxml) for each adduct entry
- Deposit archive (e.g., https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database) containing the .xlsx, .cdxml, and .mol files for all 582 adduct entries

## Outputs

- In silico MS/MS fragment lists for all adducts at three energy levels
- Interactive web datatable of predicted fragments annotated with parent adduct, filtered to fragments above 5% relative intensity

## How to apply

Extract computer-readable structure identifiers (SMILES, InChI, InChIKey, and .mol files) from the curated adduct database spreadsheet (e.g., the DNA adduct database .xlsx archive). Run the wishartlab/cfmid docker image of CFM-ID on each of the 582 structures, generating in silico MS/MS fragments at three energy levels to match the three-level scheme used for the experimental MS/MS spectra of the 15 reference standards. Filter the CFM-ID output to retain only fragment ions with relative intensity above the 5% threshold, keeping one entry per fragment ion to improve searchability. Assemble the retained fragments into an interactive web datatable using R (version 4.1.1) with the DT and Tidyverse packages, annotating each fragment with its parent adduct. Validate that the datatable contains fragment entries for all adducts at three energy levels with only fragments above 5% intensity retained.

## Related tools

- **CFM-ID** (Used as a docker image (wishartlab/cfmid) to generate in silico MS/MS fragments for all 582 DNA adduct structures at three energy levels.)
- **R (version 4.1.1)** (Used with Tidyverse packages to assemble and filter the predicted fragment datatable.)
- **DT** (R package used to render the predicted fragment datatable as an interactive web table for searchability.) — https://github.com/rstudio/DT

## Examples

```
docker run -v $(pwd):/data wishartlab/cfmid cfm-id -c /data/adduct.mol /data/out.txt
```

## Evaluation signals

- Datatable contains fragment entries for all 582 adducts at three energy levels.
- Only fragment ions with relative intensity above the 5% threshold are retained.
- One entry is kept per fragment ion to improve searchability.
- Each retained fragment is annotated with its parent adduct.
- The predicted fragment datatable is publicly available alongside the experimental spectra from the 15 reference standards.

## Limitations

- Fragments with relative intensity below 5% are discarded, so low-abundance predicted fragments are not available in the datatable.
- In silico fragments are predictions and should be used alongside, not instead of, experimental MS/MS spectra from reference standards when available.
- The three-energy-level scheme used for prediction must match the energy levels used in the experimental MS/MS spectra for comparability.

## Evidence

- [methods] Run the wishartlab/cfmid docker image of CFM-ID on each structure, generating in silico MS/MS fragments for all 582 adducts at three energy levels.: [evidence span withheld — anchor status: not_found]
- [methods] Filter to fragments above 5% intensity.: [evidence span withheld — anchor status: not_found]
- [results] Fragments above 5% intensity were included in the searchable online datatable.: [evidence span withheld — anchor status: not_found]
- [abstract] CFM-ID is used for generating in silico MS/MS fragments for all the adducts.: [evidence span withheld — anchor status: not_found]
- [readme] The DT package provides datatable() to display R data via the DataTables library.: [evidence span withheld — anchor status: external_doc]
- [abstract] The database, including 582 entries, is publicly available.: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
