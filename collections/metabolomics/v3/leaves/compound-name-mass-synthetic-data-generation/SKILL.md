---
name: compound-name-mass-synthetic-data-generation
description: Use when benchmarking TurboPutative's end-to-end processing time against
  input table size, or when synthetic compound-name/mass tables in TurboPutative's
  input format are needed to probe runtime scaling in the 500–30,000-row range.
when_to_use_negative:
- Input is a real untargeted metabolomics annotation dataset with existing putative
  annotations and feature intensities — the TPMetrics scoring/80-90% entry-reduction
  workflow, not a synthetic benchmark, is the target.
- The goal is annotation curation (Tagger classification, REname isomer/equivalent
  recognition, RowMerger redundancy removal) rather than runtime benchmarking.
- The tool under test is not TurboPutative and its input format differs from the TurboPutative
  web-service table schema.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web platform whose four modules (Tagger, REname, RowMerger, TPMetrics) are
    exercised end-to-end via its RESTful API services to measure processing time as
    a function of input table size
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:082c340c117264d1822d18f75ff6a3194d2cac069fb24dac4bf32be947d398d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Compound-name/mass synthetic data generation

## Summary

Builds synthetic TSV benchmark tables of compound-name/mass rows in TurboPutative's accepted input format (500 and 30,000 rows) and times end-to-end submission through the TurboPutative RESTful API services to characterize runtime scaling.

## When to use

Use when benchmarking TurboPutative's end-to-end processing time against input table size, or when synthetic compound-name/mass tables in TurboPutative's input format are needed to probe runtime scaling in the 500–30,000-row range.

## When NOT to use

- Input is a real untargeted metabolomics annotation dataset with existing putative annotations and feature intensities — the TPMetrics scoring/80-90% entry-reduction workflow, not a synthetic benchmark, is the target.
- The goal is annotation curation (Tagger classification, REname isomer/equivalent recognition, RowMerger redundancy removal) rather than runtime benchmarking.
- The tool under test is not TurboPutative and its input format differs from the TurboPutative web-service table schema.

## Inputs

- Compound-name/mass pairs as structured records (e.g., generated compound identifiers paired with m/z-like mass values)
- TSV table conforming to TurboPutative's accepted input format, instantiated at 500 rows and at 30,000 rows

## Outputs

- TSV benchmark table of 500 compound-name/mass rows
- TSV benchmark table of 30,000 compound-name/mass rows
- api_benchmark_timings.csv recording end-to-end processing times per table size

## How to apply

Construct TSV tables in the input format accepted by TurboPutative's web services, filling rows with compound-name/mass pairs so that row count, not annotation complexity, is the varied factor. Generate two structurally identical tables — one with 500 rows and one with 30,000 rows — to span the tested size range. Submit each table to the TurboPutative RESTful API endpoint (the platform whose workflow is exposed as RESTful services for programmed bioinformatics pipelines) via HTTP POST, and record wall-clock time from submission to completion for each run. Write the recorded timings to a results file such as api_benchmark_timings.csv. Validate against the reported near-linear scaling: ~10 s for the 500-entry table and ~250 s for the 30,000-entry table; large deviations (>~2x) indicate environmental interference (network latency, server load) rather than algorithmic non-linearity.

## Related tools

- **TurboPutative** (Web platform whose four modules (Tagger, REname, RowMerger, TPMetrics) are exercised end-to-end via its RESTful API services to measure processing time as a function of input table size) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Examples

```
curl -X POST --data-binary @table_500rows.tsv "https://proteomics.cnic.es/TurboPutative/" ; echo "$SECONDS s, $((500/SECONDS)) rows/s" >> api_benchmark_timings.csv
```

## Evaluation signals

- Measured wall-clock time for the 500-row table is approximately 10 seconds.
- Measured wall-clock time for the 30,000-row table is approximately 250 seconds.
- Ratio of timings (~25x for 60x the rows) is consistent with the reported near-linear scaling across the tested 500–30,000-row range.
- Both synthetic TSV tables are structurally identical apart from row count and parse successfully through the TurboPutative API submission without format errors.
- api_benchmark_timings.csv contains exactly two rows (one per table size) with recorded end-to-end times.

## Limitations

- Reported timings (~10 s / ~250 s) apply to TurboPutative's web services at the time of the study; remote server load and network conditions can shift absolute wall-clock times.
- Synthetic compound-name/mass rows do not carry real feature intensity or metadata, so this benchmark measures structural processing time only — not the annotation-reduction behavior (80%-90% entry reduction) that depends on genuine putative annotation content.

## Evidence

- [abstract] TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
- [abstract] allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [other] Submit the 500-row table to the TurboPutative RESTful API endpoint via HTTP POST and measure the wall-clock time from submission to completion: [evidence span withheld — anchor status: not_found]
