---
name: lc-ms-spectra-file-handling
description: Use when loading untargeted LC-MS metabolomics data (feature table plus
  MS2 fragmentation spectra) into an annotation pipeline and deciding between .mgf
  and .mzML spectral file formats.
when_to_use_negative:
- Input is raw instrument output that has not been converted to .mgf or .mzML spectra
  files or processed into a feature table
- Proteomics or genomics data — BiomiX does not yet support these omics types
- Instrument normalization of metabolomics data is required — BiomiX does not support
  it; only QC preview and removal are possible
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: BiomiX
  role: Shiny toolkit whose metabolomics module ingests .mgf/.mzML spectra and feature
    tables for annotation; Preview-QC interface handles QC visualization
  repo: https://github.com/IxI-97/BiomiX
- name: R
  role: Runtime environment in which BiomiX and its Shiny interfaces execute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:866a400746a2e7e4de217055e6297a9f1a1b8e8939947c6fc4a9dcfebff59aaf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - CEU Mass Mediator
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lc-ms-spectra-file-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare and select LC-MS spectra file formats (.mgf vs .mzML) and metadata labelling so that untargeted metabolomics annotation in BiomiX runs reliably with optimized RAM usage.

## When to use

Use when loading untargeted LC-MS metabolomics data (feature table plus MS2 fragmentation spectra) into an annotation pipeline and deciding between .mgf and .mzML spectral file formats.

## When NOT to use

- Input is raw instrument output that has not been converted to .mgf or .mzML spectra files or processed into a feature table
- Proteomics or genomics data — BiomiX does not yet support these omics types
- Instrument normalization of metabolomics data is required — BiomiX does not support it; only QC preview and removal are possible

## Inputs

- LC-MS feature table (m/z values per sample)
- MS2 fragmentation spectra in .mgf format (preferred) or .mzML
- Metadata file with CONDITION column (QC labelling) and loading-order columns

## Outputs

- Parsed spectra and feature table ready for CEU Mass Mediator / TidyMass annotation
- Preview-QC plots (scatterplots, PCA, correlation heatmaps, UMAP) with QC samples visualized before removal

## How to apply

Load the LC-MS feature table (m/z values per sample) together with the MS2 fragmentation spectra, preferring .mgf format over .mzML because .mgf files are lighter and optimize RAM usage when a high number of files is included (unlocked in BiomiX v2.4). Label QC samples by writing 'QC' in the CONDITION column of the metadata and add loading-order columns so QC samples can be visualized in PCA and UMAP and removed automatically after inspection. Use the Preview-QC Shiny interface to check scatterplots, PCA, correlation heatmaps, and UMAP before proceeding to the annotation module, which matches m/z against CEU Mass Mediator (default 15-ppm tolerance) and MS2 spectra against HMDB, MoNA, and MassBank via TidyMass. Validate that the spectra files parse correctly and that every feature's m/z enters annotation within the 15-ppm mass match error.

## Related tools

- **BiomiX** (Shiny toolkit whose metabolomics module ingests .mgf/.mzML spectra and feature tables for annotation; Preview-QC interface handles QC visualization) — https://github.com/IxI-97/BiomiX
- **R** (Runtime environment in which BiomiX and its Shiny interfaces execute)

## Examples

```
./BiomiX.sh  # open BiomiX metabolomics module; load feature table + .mgf MS2 spectra; label QC in CONDITION column for Preview-QC
```

## Evaluation signals

- Spectra files are in .mgf format (or .mzML) and parse without errors in the metabolomics module
- QC samples are visible in PCA and UMAP in the Preview-QC interface before automatic removal, confirming CONDITION-column labelling worked
- All downstream m/z matches fall within the default 15-ppm mass match error
- RAM usage stays acceptable when a high number of spectra files is included (verifying .mgf preference over .mzML)

## Limitations

- BiomiX does not support instrument normalization for metabolomics — QC samples can only be previewed and then removed
- The Mac version has installation issues; Linux or Windows is suggested
- Users must carefully review the dataset before use; no automated validation is provided

## Evidence

- [other] Load the LC-MS feature table (m/z values per sample) and the MS2 spectra in .mgf format (preferred over .mzML for lower RAM usage) into the BiomiX metabolomics module.: [evidence span withheld — anchor status: not_found]
