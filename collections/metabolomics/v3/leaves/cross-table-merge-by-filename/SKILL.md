---
name: cross-table-merge-by-filename
description: Use when per-RAW-file metadata QC tables (file-based-details TSVs from
  MetaXtract) must be combined with identification-level results to confirm or refute
  instrument underperformance signals such as depressed MS2 TIC with preserved MS1
  TIC.
when_to_use_negative:
- No identification-level results exist for the RAW files (e.g. no MaxQuant/search
  output) — only the metadata QC table can be produced, not the confirmation merge
- The filename key is not unique (multiple runs or concatenated exports share a filename)
  — the one-row-per-run invariant would break
- The RAW files are from a vendor other than Thermo Fisher, which MetaXtract does
  not yet support
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MetaXtract
  role: Produces the per-RAW-file metadata QC tables (file-based details with scan
    counts, TIC, ion injection time) that form one side of the merge
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Merges the identification counts with the per-file metadata QC tables keyed
    by RAW filename and writes the combined TSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:2750f9896cc81e5897c60fc4220ac31ebaa2592edac1812543fdf8931e68a680
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Cross-table merge by filename

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Joins search-result counts (e.g. MaxQuant peptide identifications) with MetaXtract-derived per-file acquisition metadata (scan counts, MS1/MS2 TIC, ion injection time) using the RAW filename as the key, producing a single table that tests whether metadata-derived QC signals are corroborated by identification counts.

## When to use

Use when per-RAW-file metadata QC tables (file-based-details TSVs from MetaXtract) must be combined with identification-level results to confirm or refute instrument underperformance signals such as depressed MS2 TIC with preserved MS1 TIC.

## When NOT to use

- No identification-level results exist for the RAW files (e.g. no MaxQuant/search output) — only the metadata QC table can be produced, not the confirmation merge
- The filename key is not unique (multiple runs or concatenated exports share a filename) — the one-row-per-run invariant would break
- The RAW files are from a vendor other than Thermo Fisher, which MetaXtract does not yet support

## Inputs

- MetaXtract file-based-details TSVs per RAW file (scan counts, TIC, ion injection time metrics)
- Peptide identification counts per RAW file, e.g. MaxQuant results reported for the four HeLa QC runs
- RAW filenames shared by both tables (merge key)

## Outputs

- identification_qc_confirmation.tsv — one row per run with run identifier, peptide identification count, and metadata-derived QC metrics

## How to apply

First obtain the identification counts to be checked — in the MetaXtract demonstration these are the paper-reported MaxQuant peptide counts for four Pierce HeLa Protein Digest Standard QC runs (5,055 and 5,843 on reference dates 19/12/2024 and 20/02/2025; 31 and 5 for two runs acquired on a different day). Second, load the MetaXtract file-based-details TSVs for the same four RAW files, which contain scan counts, TIC, and ion injection time metrics. Third, merge the two tables with pandas keyed on the RAW filename, taking care that the filename key resolves one-to-one (one row per run, no fan-out from duplicate names). Fourth, write the merged table as identification_qc_confirmation.tsv with one row per run holding the run identifier, peptide identification count, and metadata-derived QC metrics. The rationale is that a search-free metadata signal (depressed MS2 TIC and scan counts) should align with the search-dependent evidence (few identifications), validating MetaXtract as a near real-time, identification-free QC indicator.

## Related tools

- **MetaXtract** (Produces the per-RAW-file metadata QC tables (file-based details with scan counts, TIC, ion injection time) that form one side of the merge) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Merges the identification counts with the per-file metadata QC tables keyed by RAW filename and writes the combined TSV)

## Examples

```
import pandas as pd; merged = pd.merge(ids_df, meta_df, on='RAW_filename', validate='one_to_one'); merged.to_csv('identification_qc_confirmation.tsv', sep='\t', index=False)
```

## Evaluation signals

- Merged table has exactly one row per RAW file (four rows for the HeLa QC example) with no duplicate or unmatched keys after the merge
- Tabulated peptide identification counts match the paper's reported values exactly: 5,055 / 5,843 / 31 / 5
- Underperforming runs (31 and 5 peptides) exhibit depressed MS2 TIC and reduced scan counts while MS1 TIC is preserved, mirroring the identification deficit
- Output file identification_qc_confirmation.tsv contains the required columns: run identifier, peptide identification count, and metadata-derived QC metrics
- File-based-details input TSVs contain the MetaXtract PSI-MS-named columns (e.g. 'Total Ion Current') rather than deprecated aliases

## Limitations

- Linking QC deviations to instrument-level causes is described as plausible rather than validated by the paper
- The identification counts in this workflow are taken from the paper's reported MaxQuant results, not recomputed from search output
- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor formats are not yet supported

## Evidence
