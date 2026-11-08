---
name: public-lipidomics-dataset-retrieval
description: Use when the analysis target is a public lipidomics study identified
  by a Metabolomics Workbench study ID (e.g. ST001111) and you need to download it
  into R for inspection or reanalysis.
when_to_use_negative:
- Input is already a local Skyline CSV export or numerical matrix — use read_skyline()
  or as_lipidomics_experiment() instead of fetching from the API.
- The study of interest is not deposited on Metabolomics Workbench, so fetch_mw_study()
  cannot retrieve it.
- Molecule names have already been parsed and standardized (no ceramide-style full
  chemical names remain) — non_parsed_molecules() audit is unnecessary.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: lipidr
  role: Provides fetch_mw_study() for downloading Metabolomics Workbench studies and
    non_parsed_molecules() for auditing lipid name parsing, returning data as a LipidomicsExperiment
  repo: github.com/ahmohamed/lipidr
- name: Metabolomics Workbench API
  role: Backend REST API queried by fetch_mw_study() to search and download public
    lipidomics datasets
- name: R/Bioconductor
  role: Execution environment; lipidr is an open-source R/Bioconductor package installed
    via BiocManager
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:d26b110162aa906620f10932abe566025964dbb1c4bcf2a9a0abae4c92d62b32
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# public-lipidomics-dataset-retrieval

## Summary

Retrieves public lipidomics studies directly into R as a LipidomicsExperiment via lipidr's Metabolomics Workbench API integration (fetch_mw_study), so that public datasets like ST001111 can be inspected, reanalyzed, and audited for unparsable lipid molecule names.

## When to use

Use when the analysis target is a public lipidomics study identified by a Metabolomics Workbench study ID (e.g. ST001111) and you need to download it into R for inspection or reanalysis.

## When NOT to use

- Input is already a local Skyline CSV export or numerical matrix — use read_skyline() or as_lipidomics_experiment() instead of fetching from the API.
- The study of interest is not deposited on Metabolomics Workbench, so fetch_mw_study() cannot retrieve it.
- Molecule names have already been parsed and standardized (no ceramide-style full chemical names remain) — non_parsed_molecules() audit is unnecessary.

## Inputs

- Metabolomics Workbench study ID (e.g. ST001111)
- Network access to the Metabolomics Workbench API
- R/Bioconductor environment with lipidr installed

## Outputs

- LipidomicsExperiment object containing the fetched study's intensity matrix, sample annotations, and inferred lipid class/chain annotations
- Vector/table of non-parsed molecule names (e.g. written to non_parsed_molecules.csv)

## How to apply

Load lipidr from R/Bioconductor and call fetch_mw_study() with the study ID (e.g. "ST001111"); the function queries the Metabolomics Workbench API and returns the study as a LipidomicsExperiment, a SummarizedExperiment extension. Inspect the object's molecule names to confirm the imported lipid annotations, since lipidr automatically infers lipid class and chain information from lipid names at import. Run non_parsed_molecules() on the fetched object to enumerate lipid names that do not match lipidr's supported nomenclature patterns — in ST001111 these are ceramides written as full chemical names, which can be corrected by replacing the leading part with "Cer (" via a regular expression. Write the returned vector of non-parsed molecules to a CSV (e.g. non_parsed_molecules.csv) as an audit artifact before downstream QC, normalization, or differential analysis.

## Related tools

- **lipidr** (Provides fetch_mw_study() for downloading Metabolomics Workbench studies and non_parsed_molecules() for auditing lipid name parsing, returning data as a LipidomicsExperiment) — github.com/ahmohamed/lipidr
- **Metabolomics Workbench API** (Backend REST API queried by fetch_mw_study() to search and download public lipidomics datasets)
- **R/Bioconductor** (Execution environment; lipidr is an open-source R/Bioconductor package installed via BiocManager)

## Examples

```
library(lipidr); d <- fetch_mw_study("ST001111"); non_parsed_molecules(d)
```

## Evaluation signals

- fetch_mw_study("ST001111") returns a LipidomicsExperiment (extends SummarizedExperiment) populated with the study's samples and molecules.
- non_parsed_molecules() returns exactly the ceramide entries written as full chemical names; after regex-correcting the leading part to "Cer (", the function returns an empty result.
- Lipid class and chain information has been automatically inferred from lipid names at import (molecules carry parsed class/chain annotations).
- The audit artifact non_parsed_molecules.csv exists and matches the non-parsed molecules enumerated in the session.
- Downloaded sample names can be joined to sample annotations via add_sample_annotation() with identical sample names in both tables.

## Limitations

- Molecule names that deviate from lipidr's supported naming patterns (e.g. full-chemical-name ceramides in ST001111) fail to be parsed and require manual name correction before class/chain inference works.
- Retrieval depends on live access to the Metabolomics Workbench API; datasets must be deposited there to be fetchable.
- Sample annotation tables must use sample names identical to those in the fetched/imported data, or joining fails.

## Evidence

- [readme] lipidr represents lipidomics datasets as a LipidomicsExperiment, which extends SummarizedExperiment: [evidence span withheld — anchor status: external_doc]
- [other] Task card: non-parsed molecules in ST001111 are ceramides written with full chemical names; correctable by replacing the leading part with "Cer (": [evidence span withheld — anchor status: not_found]
