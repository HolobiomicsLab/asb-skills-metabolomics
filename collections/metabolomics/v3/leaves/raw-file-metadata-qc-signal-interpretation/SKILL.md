---
name: raw-file-metadata-qc-signal-interpretation
description: Use when RAW files from nominally identical runs (e.g. Pierce HeLa Protein
  Digest Standard QC) must be compared for instrument performance using acquisition-side
  signals alone, before or instead of database search results.
when_to_use_negative:
- Input is already a database search result (e.g. mzIdentML) and scan-level acquisition
  metrics are not needed — use search-result-based QC like MSQC instead
- RAW files are from a vendor other than Thermo Fisher — MetaXtract currently supports
  only Thermo RAW files
- MS-method or LC-method extraction is required on Linux containers — those Thermo
  method-reading calls are not supported on Linux
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Extracts file-based details (instrument details, scan counts, run statistics,
    sample information) directly from Thermo RAW files into structured TSVs used as
    the QC signal source
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Merges MetaXtract metadata QC tables with peptide identification counts by
    RAW filename to write the confirmation TSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:f239b924ef85b57ac8c9ef8022b2ffe93a9d7a35b470f120df0d485015ada8ca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Rappsilber-Laboratory/MetaXtract
  repo_url: https://github.com/Rappsilber-Laboratory/MetaXtract
  tools:
  - MetaXtract
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# raw-file-metadata-qc-signal-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets Thermo RAW-file acquisition metadata (MS1/MS2 TIC, scan counts, ion injection time) extracted by MetaXtract as search-free QC signals that indicate instrument underperformance, optionally corroborated against MaxQuant peptide identification counts (e.g. 5,055/5,843 vs 31/5 peptides across four HeLa QC runs).

## When to use

Use when RAW files from nominally identical runs (e.g. Pierce HeLa Protein Digest Standard QC) must be compared for instrument performance using acquisition-side signals alone, before or instead of database search results.

## When NOT to use

- Input is already a database search result (e.g. mzIdentML) and scan-level acquisition metrics are not needed — use search-result-based QC like MSQC instead
- RAW files are from a vendor other than Thermo Fisher — MetaXtract currently supports only Thermo RAW files
- MS-method or LC-method extraction is required on Linux containers — those Thermo method-reading calls are not supported on Linux

## Inputs

- Thermo Fisher RAW files (e.g. four Pierce HeLa Protein Digest Standard QC runs)
- MetaXtract file-based details TSV (scan counts, TIC, ion injection time metrics)
- Peptide identification counts per run from MaxQuant search results (optional, for corroboration)

## Outputs

- identification_qc_confirmation.tsv with one row per run: run identifier, peptide identification count, metadata-derived QC metrics
- Interpretation of whether metadata signals (depressed MS2 TIC, reduced scan counts, preserved MS1 TIC) corroborate underperformance

## How to apply

Run MetaXtract on the RAW files with --file-based-details to produce the file-based details TSV containing scan counts, TIC and ion injection time metrics per file. Compare these metrics across runs: underperformance is indicated by depressed MS2 TIC and scan counts while MS1 TIC is preserved, as seen when two HeLa QC runs acquired on a different day yielded only 31 and 5 peptide IDs versus 5,055 and 5,843 on reference dates (19/12/2024 and 20/02/2025). To corroborate, load MaxQuant-reported identification counts for the same RAW files and merge with the MetaXtract QC table in pandas keyed by RAW filename, writing identification_qc_confirmation.tsv with one row per run (run identifier, peptide identification count, metadata-derived QC metrics). Treat the metadata-to-cause link as plausible rather than validated: the signals are actionable indicators for troubleshooting, not mechanistic proof.

## Related tools

- **MetaXtract** (Extracts file-based details (instrument details, scan counts, run statistics, sample information) directly from Thermo RAW files into structured TSVs used as the QC signal source) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Merges MetaXtract metadata QC tables with peptide identification counts by RAW filename to write the confirmation TSV)

## Examples

```
merged = pd.merge(ident_counts, qc_details, on='RAW_filename'); merged.to_csv('identification_qc_confirmation.tsv', sep='\t', index=False)
```

## Evaluation signals

- Tabulated identification counts must match the paper's reported values exactly (5,055 / 5,843 / 31 / 5) for the four HeLa QC runs
- identification_qc_confirmation.tsv must contain exactly one row per run with the columns run identifier, peptide identification count, and metadata-derived QC metrics, keyed by RAW filename
- Underperforming runs should show reduced MS2 TIC and scan counts with preserved MS1 TIC relative to the 19/12/2024 and 20/02/2025 reference runs
- The direction of the metadata-derived QC signals must align with the identification counts demonstrating instrument underperformance

## Limitations

- Linking QC deviations to instrument-level causes is described as plausible rather than validated
- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor formats are not yet supported
- No version number or changelog is stated for MetaXtract, complicating reproducibility of the QC signal extraction
- MS-method and LC-method exports are not supported on Linux containers

## Evidence
