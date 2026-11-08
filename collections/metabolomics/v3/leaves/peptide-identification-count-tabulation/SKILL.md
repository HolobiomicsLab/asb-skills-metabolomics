---
name: peptide-identification-count-tabulation
description: Use when QC runs on the same sample (e.g. Pierce HeLa Protein Digest
  Standard) show depressed MS2 TIC or scan counts in MetaXtract metadata and you must
  confirm underperformance by comparing per-run peptide identification counts.
when_to_use_negative:
- Input data are proprietary non-Thermo vendor formats — MetaXtract currently supports
  only Thermo Fisher RAW files
- The search identification counts are unavailable and only acquisition-side metadata
  exists — use MetaXtract's search-free near-real-time monitoring instead of tabulating
  identifications
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: Extracts per-RAW-file acquisition metadata (scan counts, TIC, ion injection
    time) used as the QC signals merged with identification counts
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Merges identification counts with MetaXtract metadata TSVs keyed by RAW filename
    and writes the confirmation TSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:057c21dc53fdf6666f5c241dbae4e3c7eeb15a5d67295327bd9fc032c5dcc52d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Peptide identification count tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulates peptide identification counts from search results alongside MetaXtract-extracted acquisition QC metrics (scan counts, total ion current, ion injection time) per RAW file, to corroborate instrument underperformance signals with identification yields.

## When to use

Use when QC runs on the same sample (e.g. Pierce HeLa Protein Digest Standard) show depressed MS2 TIC or scan counts in MetaXtract metadata and you must confirm underperformance by comparing per-run peptide identification counts.

## When NOT to use

- Input data are proprietary non-Thermo vendor formats — MetaXtract currently supports only Thermo Fisher RAW files
- The search identification counts are unavailable and only acquisition-side metadata exists — use MetaXtract's search-free near-real-time monitoring instead of tabulating identifications

## Inputs

- Peptide identification counts from a database search (e.g. MaxQuant) per RAW file
- MetaXtract file-based details TSV with per-file scan counts, total ion current, and ion injection time metrics
- RAW filenames as join keys (e.g. Pierce HeLa Protein Digest Standard QC runs)

## Outputs

- identification_qc_confirmation.tsv with one row per run: run identifier, peptide identification count, metadata-derived QC metrics

## How to apply

For each QC RAW file, read the reported peptide identification count from a database search (e.g. MaxQuant searching against the UniProt human protein sequence database, as used in the paper's demonstration). Separately, load the MetaXtract file-based details TSV containing per-file scan counts, total ion current (MS:1000285), and ion injection time (MS:1000927) metrics. Merge the two tables with pandas, keyed by RAW filename, into a single TSV with one row per run holding run identifier, identification count, and metadata-derived QC metrics. Decide whether underperformance is corroborated: in the paper, reference HeLa QC runs identified 5,055 and 5,843 peptides while two runs acquired on a different day identified only 31 and 5, with matching depressed MS2 TIC but preserved MS1 TIC.

## Related tools

- **MetaXtract** (Extracts per-RAW-file acquisition metadata (scan counts, TIC, ion injection time) used as the QC signals merged with identification counts) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Merges identification counts with MetaXtract metadata TSVs keyed by RAW filename and writes the confirmation TSV)

## Examples

```
python main.py --config config.yml  # MetaXtract run; then merge: pd.merge(ids_df, meta_df, on='raw_file').to_csv('identification_qc_confirmation.tsv', sep='\t', index=False)
```

## Evaluation signals

- Tabulated identification counts match the search output exactly (e.g. 5,055 / 5,843 / 31 / 5 in the paper's HeLa QC demonstration)
- Every RAW file that has an identification count also has a row in the merged TSV (join key = RAW filename, no dropped runs)
- Underperforming runs show both low identification counts and depressed metadata QC signals (MS2 TIC, scan counts) with preserved MS1 TIC
- Output TSV schema contains run identifier, peptide identification count, and metadata-derived QC metric columns

## Limitations

- Linking QC deviations to instrument-level causes is described as plausible rather than validated
- Identification counts depend on a completed database search; MetaXtract's own signals are search-free and available immediately after acquisition
- MetaXtract supports only Thermo Fisher RAW files; additional vendor formats not yet supported

## Evidence
