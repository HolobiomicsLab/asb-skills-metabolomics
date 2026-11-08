---
name: odd-chain-lipid-quality-check
description: Use when preprocessing vendor lipid identification output (e.g., LipidSearch
  or LIQUID tables) and you must validate parsed lipid species — total carbons, double
  bonds, odd-chain flagging — before normalization.
when_to_use_negative:
- Input is already a normalized/feature table (e.g., a SummarizedExperiment concentration
  matrix) rather than raw vendor identification output — re-filtering would double-process
  species
- Data originate from the Metabolomics Workbench download path (do_query()) where
  lipid identifiers may already conform to expected conventions rather than needing
  vendor-output boundary filtering
- The experiment lacks lipid-species identifiers amenable to LIPID MAPS classification
  parsing, so carbon/double-bond boundaries cannot be evaluated
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ADViSELipidomics
  role: 'Shiny app implementing the preprocessing module: LIPID MAPS parsing, boundary
    filtering, odd-chain flagging, and internal-standard recovery normalization'
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Vendor tool whose output files are the input identifiers being parsed and
    boundary-filtered
- name: LIPID MAPS
  role: Classification scheme used to parse lipid species into class, carbon-atom
    count, and double-bond count
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:6830f954b2537f6076ab808696cd965789a9ee8e99d5dbb48638b076d36f6455
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIPID MAPS
  - LipidSearch
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# odd-chain-lipid-quality-check

## Summary

A lipid-identifier quality-control step from the ADViSELipidomics preprocessing module: parse lipid species names using the LIPID MAPS classification (class, total carbon atoms, double-bond count), filter the lipid list against class-specific boundaries in an internal reference file, and flag odd-chain species while removing duplicates and boundary violations.

## When to use

Use when preprocessing vendor lipid identification output (e.g., LipidSearch or LIQUID tables) and you must validate parsed lipid species — total carbons, double bonds, odd-chain flagging — before normalization.

## When NOT to use

- Input is already a normalized/feature table (e.g., a SummarizedExperiment concentration matrix) rather than raw vendor identification output — re-filtering would double-process species
- Data originate from the Metabolomics Workbench download path (do_query()) where lipid identifiers may already conform to expected conventions rather than needing vendor-output boundary filtering
- The experiment lacks lipid-species identifiers amenable to LIPID MAPS classification parsing, so carbon/double-bond boundaries cannot be evaluated

## Inputs

- Vendor lipid identification/quantification output (LipidSearch or LIQUID files, e.g., Case Study 1 example data)
- Internal reference file with per-lipid-class boundaries on carbon-atom and double-bond counts
- Internal lipid standard identities (for downstream recovery normalization)

## Outputs

- Filtered lipid list with boundary-violating species removed, duplicates dropped, and odd-chain species flagged
- Parsed lipid annotations (class, total carbon atoms, double-bond count) per species
- Cleaned concentration matrix (one row per retained lipid species, one column per sample) assembled into a SummarizedExperiment object

## How to apply

Parse each lipid species identifier using the LIPID MAPS classification to extract the lipid class, total carbon-atom count, and double-bond count. Load the internal reference file that stores per-class boundaries on carbon atoms and double bonds, then remove lipid species that violate these limits and drop duplicate entries; species with odd carbon-atom counts are flagged (retained but marked) rather than silently deleted. This filtering must run before internal-standard recovery normalization, because the corrected abundance matrix should contain one row per retained lipid species. The cleaned lipid list is then carried into the normalization step, which computes recovery percentages from spiked internal lipid standards and assembles a SummarizedExperiment with sample metadata.

## Related tools

- **ADViSELipidomics** (Shiny app implementing the preprocessing module: LIPID MAPS parsing, boundary filtering, odd-chain flagging, and internal-standard recovery normalization) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Vendor tool whose output files are the input identifiers being parsed and boundary-filtered)
- **LIPID MAPS** (Classification scheme used to parse lipid species into class, carbon-atom count, and double-bond count)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- Output file exists and loads as a SummarizedExperiment object
- Concentration matrix is non-empty with one row per retained lipid species and one column per sample
- No retained species violates the carbon-atom or double-bond boundaries defined in the internal reference file
- Duplicate lipid species entries are absent from the filtered list
- Odd-chain species are present only as flagged entries, not removed, and normalized values reflect applied internal-standard recovery percentages

## Limitations

- Filtering depends on the completeness of the internal reference file boundaries; species outside the encoded class limits are removed even if genuinely detected
- The batch correction method used alongside this preprocessing is not explicitly named in the paper's discussion, so its interaction with filtering is undocumented
- Power to detect non-linear drift in replicate QC samples is low because variance is very low, so QC-driven re-filtering decisions are weakly supported

## Evidence

- [readme] LIPID MAPS parsing of species identifiers underpins the quality check: "ADViSELipidomics extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples"
- [readme] Internal-standard normalization follows the filtered matrix: "In the presence of internal lipid standards in the experiment, ADViSELipidomics can normalize the data matrix, providing absolute values of concentration per lipid and sample."
- [intro] Vendor outputs handled by the preprocessing workflow: "It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification"
- [abstract] Preprocessing scope stated in the abstract: "ADViSELipidomics is a novel Shiny app for preprocessing, analyzing and visualizing lipidomics data."
