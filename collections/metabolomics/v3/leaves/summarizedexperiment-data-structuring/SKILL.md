---
name: summarizedexperiment-data-structuring
description: Use when vendor lipid quantification output (LipidSearch or LIQUID, or
  Metabolomics Workbench data) has been parsed, boundary-filtered, and internal-standard
  normalized, and needs to be packaged with sample metadata into a standard Bioconductor
  container for downstream analysis.
when_to_use_negative:
- Input data has not yet been preprocessed — e.g. raw LipidSearch/LIQUID vendor output
  that still needs lipid identifier parsing and boundary filtering before normalization
  and structuring.
- The analysis goal is batch effect correction or differential abundance testing on
  an existing SummarizedExperiment — those are downstream skills that consume, rather
  than build, this container.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: Shiny app that performs the preprocessing (parsing, boundary filtering, internal-standard
    recovery normalization) and assembles the concentration matrix with sample metadata
    into a SummarizedExperiment object
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Vendor software whose quantification output files are the upstream source
    for the concentration matrix being structured
- name: LIPID MAPS
  role: Classification used to parse lipid species identifiers into class, carbon-atom
    count, and double-bond count prior to filtering and structuring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:9ed6279beaefef62f957cd35af8a6ef7100cd2721695faee5b715a4c925575d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIPID MAPS
  - LipidSearch
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# SummarizedExperiment data structuring

## Summary

Assemble the corrected, internal-standard-normalized lipid concentration matrix together with sample metadata into an R SummarizedExperiment object, the container ADViSELipidomics uses to hand preprocessed lipidomics data to downstream exploratory and statistical analyses.

## When to use

Use when vendor lipid quantification output (LipidSearch or LIQUID, or Metabolomics Workbench data) has been parsed, boundary-filtered, and internal-standard normalized, and needs to be packaged with sample metadata into a standard Bioconductor container for downstream analysis.

## When NOT to use

- Input data has not yet been preprocessed — e.g. raw LipidSearch/LIQUID vendor output that still needs lipid identifier parsing and boundary filtering before normalization and structuring.
- The analysis goal is batch effect correction or differential abundance testing on an existing SummarizedExperiment — those are downstream skills that consume, rather than build, this container.

## Inputs

- Corrected and normalized lipid concentration matrix (abundance values scaled by internal-standard recovery percentages), one row per retained lipid species and one column per sample
- Sample metadata available in the experiment
- Filtered lipid species table (parsed via LIPID MAPS classification: lipid class, total carbon atoms, double-bond count)

## Outputs

- SummarizedExperiment object containing a non-empty concentration matrix (rows = retained lipid species, columns = samples) with normalized values, plus sample metadata
- Downstream-exportable normalized concentration output file

## How to apply

After parsing lipid species identifiers via LIPID MAPS classification (class, total carbon atoms, double-bond count) and filtering the lipid list against the internal reference boundaries (removing species violating double-bond or carbon-atom limits and duplicates, flagging odd-chain species), compute the recovery percentage of the spiked internal lipid standards per sample and use these recoveries to scale lipid abundances into absolute concentration values. Assemble the resulting normalized corrected concentration matrix — one row per retained lipid species, one column per sample — with the available sample metadata into a SummarizedExperiment object inside the ADViSELipidomics Shiny app. The SummarizedExperiment structure keeps abundances and sample information aligned so downstream steps (exploratory plots, differential abundance, batch effect correction) operate on a single coherent object. Note that when no internal standards are present, the normalization branch that produces absolute concentrations is skipped and the structuring applies to the non-normalized matrix instead.

## Related tools

- **ADViSELipidomics** (Shiny app that performs the preprocessing (parsing, boundary filtering, internal-standard recovery normalization) and assembles the concentration matrix with sample metadata into a SummarizedExperiment object) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Vendor software whose quantification output files are the upstream source for the concentration matrix being structured)
- **LIPID MAPS** (Classification used to parse lipid species identifiers into class, carbon-atom count, and double-bond count prior to filtering and structuring)

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- The output file exists and loads as a SummarizedExperiment object in R.
- The concentration matrix is non-empty, with exactly one row per retained lipid species and one column per sample.
- Matrix values are normalized (internal-standard recovery-corrected) concentration values, not raw vendor intensities.
- Sample metadata stored in the object matches the columns of the concentration matrix.
- Rows correspond to the lipid species retained after boundary filtering (no duplicates, no species violating double-bond or carbon-atom limits).

## Limitations

- Normalization to absolute concentrations via internal-standard recovery percentages is only available when internal lipid standards are included in the experiment; otherwise the concentration matrix cannot be produced in normalized form.
- ADViSELipidomics is marked experimental/stable-lifecycle software; interface layout varies with monitor resolution and zoom, and the Docker version is tested on Chrome (Firefox can have problems).
- Batch correction details are not fully identified in the paper, so the downstream steps consuming this object may have undocumented method choices.

## Evidence

- [supplementary] For each sample, compute the recovery percentage of the spiked internal lipid standards and use these recoveries to scale lipid abundances into absolute concentration values.: [evidence span withheld — anchor status: not_found]
- [supplementary] Assemble the corrected and normalized concentration matrix with sample metadata into a SummarizedExperiment object.: [evidence span withheld — anchor status: not_found]
- [readme] In the presence of internal lipid standards in the experiment, ADViSELipidomics can normalize the data matrix, providing absolute values of concentration per lipid and sample.: "In the presence of internal lipid standards in the experiment, ADViSELipidomics
can normalize the data matrix, providing absolute values of
concentration per lipid and sample."
- [abstract] When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples.: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [intro] extracts information by parsing lipid species (using LIPID MAPS classification) and, together with information available on the samples, allows performing several exploratory and statistical analyses: "extracts information by parsing lipid species (using LIPID MAPS
classification) and, together with information available on the samples,
allows performing several exploratory and statistical analyses"
