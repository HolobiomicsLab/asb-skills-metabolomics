---
name: internal-standard-recovery-normalization
description: Use when lipidomics quantification output (e.g. LipidSearch or LIQUID
  tables) contains spiked internal lipid standards and the goal is a normalized, absolute
  concentration matrix per lipid and sample.
when_to_use_negative:
- The experiment contains no internal lipid standards — without spikes no recovery
  percentage can be computed and this normalization does not apply.
- The input is not a LipidSearch/LIQUID lipid identification and quantification output
  (ADVISELipidomics also supports Metabolomics Workbench data, but this recovery-based
  normalization targets vendor lipidomics tables).
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ADViSELipidomics
  role: Shiny app that computes internal-standard recovery percentages and applies
    them to produce the normalized absolute concentration matrix
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Vendor software whose lipid identification/quantification output is the input
    to the normalization
- name: LIQUID
  role: Alternative vendor software whose output can serve as input to the normalization
- name: metabolomicsWorkbenchR
  role: R package used to download suitable lipidomic experiments (via do_query())
    as input data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:8a354f530c4d756434733e4a6851c9831c65cf6ffbe071056630be6d49515acd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIQUID
  - LipidSearch
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# internal-standard-recovery-normalization

## Summary

ADViSELipidomics normalizes the lipid data matrix when internal lipid standards are spiked into the experiment, computing a recovery percentage per sample from the internal standards and applying it to each lipid to obtain normalized corrected concentrations (absolute abundance values).

## When to use

Use when lipidomics quantification output (e.g. LipidSearch or LIQUID tables) contains spiked internal lipid standards and the goal is a normalized, absolute concentration matrix per lipid and sample.

## When NOT to use

- The experiment contains no internal lipid standards — without spikes no recovery percentage can be computed and this normalization does not apply.
- The input is not a LipidSearch/LIQUID lipid identification and quantification output (ADVISELipidomics also supports Metabolomics Workbench data, but this recovery-based normalization targets vendor lipidomics tables).

## Inputs

- LipidSearch or LIQUID vendor output files (lipid identification and quantification tables)
- Experiment metadata indicating the presence and identity of spiked internal lipid standards
- Sample metadata (e.g. from Metabolomics Workbench experiments via metabolomicsWorkbenchR do_query())

## Outputs

- SummarizedExperiment object holding a normalized corrected concentration matrix (one row per retained lipid species, one column per sample)
- Per-sample internal-standard recovery percentages

## How to apply

Load the vendor output (e.g. LipidSearch files from Case Study 1) into the ADViSELipidomics Shiny app, which parses lipid species identifiers using LIPID MAPS classification to extract class, carbon-atom count, and double-bond count. With internal lipid standards present, the app computes the recovery percentage of the spiked internal standards for each sample and scales every lipid's abundance by these recoveries, converting relative abundances into absolute concentration values. The corrected values are assembled, together with sample metadata, into a SummarizedExperiment object containing the normalized concentration matrix. Validation: the output exists, loads as a SummarizedExperiment, and has a non-empty matrix with one row per retained lipid species and one column per sample holding normalized values.

## Related tools

- **ADViSELipidomics** (Shiny app that computes internal-standard recovery percentages and applies them to produce the normalized absolute concentration matrix) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Vendor software whose lipid identification/quantification output is the input to the normalization)
- **LIQUID** (Alternative vendor software whose output can serve as input to the normalization)
- **metabolomicsWorkbenchR** (R package used to download suitable lipidomic experiments (via do_query()) as input data)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- Output file exists and loads as a SummarizedExperiment object.
- Concentration matrix is non-empty with one row per retained lipid species and one column per sample.
- Matrix values are normalized (corrected concentrations), not raw vendor abundances.
- Each sample's scaling factor corresponds to the computed recovery percentage of its spiked internal lipid standards.

## Limitations

- Normalization to absolute concentrations is only possible when internal lipid standards are included in the experiment.
- If needed to install many packages from source, installation/compilation can take a long time depending on hardware and OS (relevant to running the app that performs this step).
- The Shiny interface appearance depends on screen size/resolution and may need zoom adjustment to interact with the normalization step.

## Evidence

- [abstract] When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples.: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [readme] In the presence of internal lipid standards in the experiment, ADViSELipidomics can normalize the data matrix, providing absolute values of concentration per lipid and sample.: "In the presence of internal lipid standards in the experiment, ADViSELipidomics
can normalize the data matrix, providing absolute values of
concentration per lipid and sample."
- [supplementary] For each sample, compute the recovery percentage of the spiked internal lipid standards and use these recoveries to scale lipid abundances into absolute concentration values.: [evidence span withheld — anchor status: not_found]
- [supplementary] Assemble the corrected and normalized concentration matrix with sample metadata into a SummarizedExperiment object.: [evidence span withheld — anchor status: not_found]
