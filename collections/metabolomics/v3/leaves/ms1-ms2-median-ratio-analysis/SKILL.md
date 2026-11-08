---
name: ms1-ms2-median-ratio-analysis
description: Use when comparing HeLa Pierce QC RAW runs to a reference set to diagnose
  whether peptide-identification loss stems from MS1 TIC decline or an MS2 isolation/fragment-transmission
  failure.
when_to_use_negative:
- Input data has not been through MetaXtract scan-header export — the 'Total Ion Current'
  column and MS1/MS2 scan separation come from that tool, not from database search
  results
- The comparison requires identification-level metrics (e.g. peptide counts); this
  skill uses only acquisition-side signals and cannot attribute causes requiring search
  results
- RAW files are not from Thermo instruments — MetaXtract currently supports only Thermo
  Fisher RAW files
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: Extracts MS1 and MS2 scan-header tables (including 'Total Ion Current', MS:1000285)
    directly from Thermo RAW files; pandas is then used to compute per-run medians
    and ratios
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Aggregates per-scan Total Ion Current values into per-run medians and computes
    the underperforming-vs-reference ratios exported to qc_tic_comparison.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:98afa0586512bbd341261add190ae0754a4360434acd9443fd061b9d80f8d7f5
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ms1-ms2-median-ratio-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-run median Total Ion Current from MetaXtract-exported MS1 and MS2 scan headers and take underperforming-vs-reference ratios (e.g. 8.4% MS1 drop, 9.2-fold MS2 reduction) to localize acquisition underperformance without database searching.

## When to use

Use when comparing HeLa Pierce QC RAW runs to a reference set to diagnose whether peptide-identification loss stems from MS1 TIC decline or an MS2 isolation/fragment-transmission failure.

## When NOT to use

- Input data has not been through MetaXtract scan-header export — the 'Total Ion Current' column and MS1/MS2 scan separation come from that tool, not from database search results
- The comparison requires identification-level metrics (e.g. peptide counts); this skill uses only acquisition-side signals and cannot attribute causes requiring search results
- RAW files are not from Thermo instruments — MetaXtract currently supports only Thermo Fisher RAW files

## Inputs

- MS1 scan-header TSV/CSV exported by MetaXtract with 'Total Ion Current' column
- MS2 scan-header TSV/CSV exported by MetaXtract with 'Total Ion Current' column
- Mapping of RAW runs to underperforming (UP) vs reference groups

## Outputs

- qc_tic_comparison.csv containing per-run median MS1 and MS2 Total Ion Current, the MS1 relative decrease ratio, and the MS2 reduction factor
- Median MS1 TIC relative decrease (expected 8.4%)
- Median MS2 TIC reduction factor (expected 9.2-fold)

## How to apply

Load the MS1 and MS2 scan-header tables exported by MetaXtract, each containing the 'Total Ion Current' column per scan (PSI-MS term MS:1000285). For each run compute the median Total Ion Current separately over MS1 and MS2 scans. Take the ratio of median MS1 TIC in the underperforming runs to the median MS1 TIC in reference runs to report the relative decrease (expected 8.4%), then compute the median MS2 TIC reduction factor analogously (expected 9.2-fold). A large MS1 drop with a much larger MS2 drop indicates an isolation/fragment-transmission problem rather than a loading failure, since MS2 signal should roughly track MS1 intensity under identical acquisition settings. Aggregate per-run medians plus both ratio metrics into a summary table and export as qc_tic_comparison.csv.

## Related tools

- **MetaXtract** (Extracts MS1 and MS2 scan-header tables (including 'Total Ion Current', MS:1000285) directly from Thermo RAW files; pandas is then used to compute per-run medians and ratios) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Aggregates per-scan Total Ion Current values into per-run medians and computes the underperforming-vs-reference ratios exported to qc_tic_comparison.csv)

## Examples

```
python main.py --config /path/to/config.yml  # with MS1/MS2 scan headers (Total Ion Current: true), then: pd.read_csv('...ms1_*.csv').groupby('run')['Total Ion Current'].median()
```

## Evaluation signals

- MS1 median TIC ratio of underperforming to reference runs equals the reported 8.4% relative decrease (Figure 2a)
- MS2 median TIC reduction factor equals the reported 9.2-fold decrease (Figure 2b)
- Medians are computed per run, not pooled across runs, and are derived only from scan-header Total Ion Current values
- Output qc_tic_comparison.csv contains one row per run plus the two ratio metrics
- No database-search results were used at any step (search-free QC)

## Limitations

- Linking QC deviations to instrument-level causes is described as plausible rather than validated — the 9.2-fold MS2 drop points to isolation/fragment transmission but the article labels the mechanism 'plausible'
- The analysis is diagnostic only: it does not by itself correct acquisition or recover identifications
- Currently supports only Thermo Fisher RAW files, since MetaXtract reads them via RawFileReader

## Evidence

- [results] MS1 TIC was 8.4% lower in UP runs, while MS2 signal intensity was 9.2-fold reduced, indicating an isolation/fragment transmission problem as the primary cause of peptide identification loss.: [evidence span withheld — anchor status: not_found]
