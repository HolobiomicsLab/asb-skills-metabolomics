---
name: lipid-species-annotation-parsing
description: Use when raw lipid identification/quantification files from LipidSearch
  or LIQUID (or a Metabolomics Workbench experiment) must be converted into a structured
  table of lipid class, chain length, and double-bond strata before normalization
  or differential analysis.
when_to_use_negative:
- Input is already a parsed, boundary-filtered abundance/concentration matrix rather
  than raw vendor (LipidSearch/LIQUID) identification and quantification output
- The experiment data are not in a format ADViSELipidomics handles (LipidSearch, LIQUID,
  or a Metabolomics Workbench experiment retrievable via the metabolomicsWorkbenchR
  do_query() function)
- The goal is purely statistical analysis of an existing parsed table with no need
  to re-stratify species into class/chain/double-bond fields
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ADViSELipidomics
  role: Shiny app that performs the data import, feature pre-filtering, and LIPID
    MAPS-based lipid species parsing
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Vendor software whose identification and quantification output files are the
    parsed input
- name: LIPID MAPS
  role: Classification scheme used to stratify each lipid species into class, chain
    length, and double-bond count
- name: metabolomicsWorkbenchR
  role: R package whose do_query() function downloads Metabolomics Workbench lipidomic
    experiments as alternative input
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:a7662f795e8a21ca6c9afc8e984890ea33cdbfa5ae62d5b13297f912e5b24cef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIPID MAPS
  - LipidSearch
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lipid-species-annotation-parsing

## Summary

Parses lipid species identifiers from vendor lipidomics outputs (LipidSearch, LIQUID, Metabolomics Workbench) using the LIPID MAPS classification, stratifying each species into class, chain length/carbon-atom count, and double-bond count, and pre-filters species against internal reference boundaries before downstream normalization.

## When to use

Use when raw lipid identification/quantification files from LipidSearch or LIQUID (or a Metabolomics Workbench experiment) must be converted into a structured table of lipid class, chain length, and double-bond strata before normalization or differential analysis.

## When NOT to use

- Input is already a parsed, boundary-filtered abundance/concentration matrix rather than raw vendor (LipidSearch/LIQUID) identification and quantification output
- The experiment data are not in a format ADViSELipidomics handles (LipidSearch, LIQUID, or a Metabolomics Workbench experiment retrievable via the metabolomicsWorkbenchR do_query() function)
- The goal is purely statistical analysis of an existing parsed table with no need to re-stratify species into class/chain/double-bond fields

## Inputs

- LipidSearch identification and quantification output files (also supports LIQUID outputs)
- Target file with sample metadata accompanying the vendor outputs
- Internal reference file defining double-bond and carbon-atom boundaries per lipid class
- Case Study 1 example archive (Case_Study_1.zip from the ADViSELipidomics_book data_example repository)

## Outputs

- parsed_lipid_species.csv — one row per lipid species with columns for class, chain length, and double-bond count
- Feature matrix pre-filtered to features of interest
- Normalized corrected abundance/concentration matrix (one row per retained lipid species, one column per sample), optionally as a SummarizedExperiment object

## How to apply

Load the vendor identification and quantification files (e.g., the LipidSearch output files of Case Study 1, available as Case_Study_1.zip from the ShinyFabio/ADViSELipidomics_book data_example repository) together with the accompanying target file into the ADViSELipidomics Shiny app (R >= 4.0, launched via run_ADViSELipidomics()). During the import stage, apply the feature pre-filtering to remove non-informative entries, then parse each lipid species identifier with the LIPID MAPS classification embedded in ADViSELipidomics, extracting lipid class, total carbon atoms/chain length, and double-bond count as separate strata. Filter the lipid list against the boundaries in the internal reference file, removing species that violate double-bond or carbon-atom limits and duplicate entries, and flagging odd-chain species. When internal lipid standards are present, compute per-sample recovery percentages of the spiked standards and scale abundances into absolute concentration values. Assemble the corrected, normalized concentration matrix with sample metadata (e.g., into a SummarizedExperiment object) for downstream exploratory and statistical analyses.

## Related tools

- **ADViSELipidomics** (Shiny app that performs the data import, feature pre-filtering, and LIPID MAPS-based lipid species parsing) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Vendor software whose identification and quantification output files are the parsed input)
- **LIPID MAPS** (Classification scheme used to stratify each lipid species into class, chain length, and double-bond count)
- **metabolomicsWorkbenchR** (R package whose do_query() function downloads Metabolomics Workbench lipidomic experiments as alternative input)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- parsed_lipid_species.csv exists, is valid CSV, and contains the class/chain/double-bond fields for every lipid species in the input
- No lipid species failed LIPID MAPS classification (all species resolved into class, chain length, and double bonds)
- Species violating double-bond or carbon-atom limits, duplicate entries, and odd-chain species are removed or flagged per the internal reference boundaries
- Output concentration matrix is non-empty with one row per retained lipid species and one column per sample, containing normalized values
- Output loads as a SummarizedExperiment object when sample metadata is assembled

## Limitations

- Odd-chain species are only flagged during boundary filtering rather than silently retained, and species violating carbon-atom or double-bond limits are removed — check flags before assuming a species was lost to error
- The batch correction step used for the data is not identified by method name in the paper, so downstream batch-effect handling cannot be reproduced exactly
- Pre-filters are defined by the internal reference file boundaries, so lipid classes outside those boundaries will be dropped rather than parsed

## Evidence

- [abstract] abstract: "extracts information by parsing lipid species (using LIPID MAPS classification)"
- [readme] readme: "ADViSELipidomics extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples,"
- [readme] readme-standards: "In the presence of internal lipid standards in the experiment, ADViSELipidomics can normalize the data matrix, providing absolute values of concentration per lipid and sample."
- [abstract] abstract-vendor: "It handles the outputs from LipidSearch and LIQUID for lipid identification and quantification"
