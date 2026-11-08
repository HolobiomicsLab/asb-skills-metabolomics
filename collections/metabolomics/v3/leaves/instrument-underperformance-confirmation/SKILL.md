---
name: instrument-underperformance-confirmation
description: Use when a Thermo RAW run shows depressed acquisition-side QC signals
  (e.g. reduced MS2 TIC with preserved MS1 TIC) and you need identification-count
  evidence from MaxQuant-style searches to confirm the underperformance.
when_to_use_negative:
- Input is a single run without a paired reference underperforming/control set — the
  corroboration logic needs both reference and underperforming identification counts
- The task is extracting metadata itself (use MetaXtract's file-based-details workflow
  directly) rather than cross-checking identification counts against already-extracted
  metadata
- The RAW file is not from a Thermo instrument, since MetaXtract currently supports
  only Thermo Fisher RAW files
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: Extracts the file-based-details TSV metadata tables (scan counts, TIC, ion
    injection time) from the Thermo RAW files used to corroborate identification-count
    evidence
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Merges the identification counts with per-file metadata QC signals keyed by
    RAW filename and writes the confirmation TSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:dfbf7b6f3e282a21496fccfbb67de12799edeecf853106fb37db6ce3d189ade9
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# instrument-underperformance-confirmation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Confirms suspected instrument underperformance by cross-checking peptide identification counts from database searches against acquisition-side metadata QC signals (MS2 TIC, scan counts, ion injection time) extracted by MetaXtract from Thermo RAW files.

## When to use

Use when a Thermo RAW run shows depressed acquisition-side QC signals (e.g. reduced MS2 TIC with preserved MS1 TIC) and you need identification-count evidence from MaxQuant-style searches to confirm the underperformance.

## When NOT to use

- Input is a single run without a paired reference underperforming/control set — the corroboration logic needs both reference and underperforming identification counts
- The task is extracting metadata itself (use MetaXtract's file-based-details workflow directly) rather than cross-checking identification counts against already-extracted metadata
- The RAW file is not from a Thermo instrument, since MetaXtract currently supports only Thermo Fisher RAW files

## Inputs

- Four Pierce HeLa Protein Digest Standard QC Thermo RAW files (two reference, two underperforming)
- Reported MaxQuant peptide identification counts from the paper (5,055, 5,843, 31, 5)
- MetaXtract file-based-details TSV containing scan counts, TIC, and ion injection time metrics per RAW file

## Outputs

- identification_qc_confirmation.tsv with one row per run: run identifier, peptide identification count, metadata-derived QC metrics

## How to apply

Extract the four HeLa QC identification counts reported in the paper (5,055 and 5,843 for reference runs on 19/12/2024 and 20/02/2025; 31 and 5 for the underperforming runs on a different acquisition day) and load the MetaXtract file-based-details TSV tables for the same RAW files, which contain scan counts, TIC, and ion injection time metrics. Merge both sources into a single TSV keyed by RAW filename using pandas, producing `identification_qc_confirmation.tsv` with one row per run. Validate that the tabulated identification counts match the reported values exactly and that the depressed identification counts co-occur with the metadata QC signals (reduced MS2 TIC with preserved MS1 TIC), establishing corroborating evidence for instrument underperformance rather than biological variation.

## Related tools

- **MetaXtract** (Extracts the file-based-details TSV metadata tables (scan counts, TIC, ion injection time) from the Thermo RAW files used to corroborate identification-count evidence) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Merges the identification counts with per-file metadata QC signals keyed by RAW filename and writes the confirmation TSV)

## Examples

```
python main.py --config config.yml; then merge MetaXtract file-based-details TSV with reported peptide counts via pandas into identification_qc_confirmation.tsv
```

## Evaluation signals

- Tabulated identification counts must equal the paper's reported values exactly (5,055 / 5,843 / 31 / 5)
- Each underperforming run row must show both a drastically low identification count (≤31) and a depressed MS2 TIC signal while MS1 TIC remains comparable
- identification_qc_confirmation.tsv must contain exactly one row per RAW file, keyed by filename, with no missing identifiers
- Metadata QC metrics in the TSV must be traceable back to the MetaXtract file-based-details extraction of the same RAW file

## Limitations

- The paper states linking QC deviations to instrument-level causes is plausible rather than fully validated; the confirmation rests on corroboration, not causal proof
- The demonstration dataset is the Pierce HeLa Protein Digest Standard acquired on an Orbitrap with nano LC DDA — generalization to other samples, instruments, or acquisition modes is not established
- Identification counts depend on downstream database-search parameters (UniProt human database, MaxQuant), so differing search settings would alter the counts and break the expected values

## Evidence
