---
name: metabolomics-input-table-formatting
description: Use when preparing TSV compound-annotation tables as TurboPutative inputs
  and confirming end-to-end processing time scales acceptably with table size before
  submitting larger untargeted-metabolomics datasets.
when_to_use_negative:
- Input is already TurboPutative output (annotation entries already reduced by 80%-90%)
  that needs downstream interpretation or pathway prioritization rather than formatting
- The goal is metabolite classification or TPMetrics scoring itself, not preparing/benchmarking
  the input table for API submission
- Datasets far outside the tested 500-30,000-row range, where the reported near-linear
  timing expectation was not validated
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TurboPutative
  role: Web platform (Tagger, REname, RowMerger, TPMetrics modules) whose RESTful
    API ingests the formatted TSV tables and whose wall-clock processing times are
    measured for the scaling check
  repo: CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:9f7d145f09815e7de448ac4e7ac2a2687cf5bb62b5514c02c9ead6a5dd298b0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolomics-input-table-formatting

## Summary

Formats compound annotation tables (TSV with compound-name/mass rows) as input for TurboPutative's RESTful API services and verifies the pipeline's near-linear runtime scaling (~10 s for 500 rows, ~250 s for 30,000 rows) before batch processing.

## When to use

Use when preparing TSV compound-annotation tables as TurboPutative inputs and confirming end-to-end processing time scales acceptably with table size before submitting larger untargeted-metabolomics datasets.

## When NOT to use

- Input is already TurboPutative output (annotation entries already reduced by 80%-90%) that needs downstream interpretation or pathway prioritization rather than formatting
- The goal is metabolite classification or TPMetrics scoring itself, not preparing/benchmarking the input table for API submission
- Datasets far outside the tested 500-30,000-row range, where the reported near-linear timing expectation was not validated

## Inputs

- TSV table of compound-name/mass rows accepted by TurboPutative's web services
- Structurally identical TSV tables at two sizes for the scaling check (e.g. 500 rows and 30,000 rows)

## Outputs

- Formatted TSV input tables ready for TurboPutative RESTful API submission
- api_benchmark_timings.csv recording end-to-end processing times per table size

## How to apply

Build a TSV input table with compound-name/mass rows conforming to the format accepted by TurboPutative's web services (Tagger, REname, RowMerger, TPMetrics). First submit a small 500-row table to the TurboPutative RESTful API endpoint via HTTP POST and record wall-clock time from submission to completion; then repeat with a structurally identical 30,000-row table. Acceptance is based on the reported benchmark: approximately 10 s for 500 rows and approximately 250 s for 30,000 rows, consistent with near-linear scaling (runtime ratio ≈ 25× for a 60× size increase). If measured times deviate substantially from this scaling, investigate before committing larger batch runs. Write the recorded times to api_benchmark_timings.csv as the audit artifact.

## Related tools

- **TurboPutative** (Web platform (Tagger, REname, RowMerger, TPMetrics modules) whose RESTful API ingests the formatted TSV tables and whose wall-clock processing times are measured for the scaling check) — CNIC-Proteomics/TurboPutative-web

## Examples

```
import requests, time; t=time.time(); r=requests.post('https://proteomics.cnic.es/TurboPutative/', files={'file': open('table_500.tsv','rb')}); print(time.time()-t)
```

## Evaluation signals

- Measured end-to-end time for the 500-row TSV is approximately 10 s
- Measured end-to-end time for the 30,000-row TSV is approximately 250 s
- Runtime ratio between the two tables is consistent with near-linear scaling (~25× for a 60× row increase)
- api_benchmark_timings.csv exists and contains one recorded wall-clock time per submitted table size
- Both API submissions complete successfully (no malformed-input rejection of the TSV format)

## Limitations

- Timing validation reported only within the 500-30,000-row tested range; scaling behavior outside this range is not established
- The source document contains no discussion section or changelog, so no additional tuning guidance or failure-mode notes are available beyond the benchmark figures

## Evidence

- [abstract] TurboPutative RESTful API submission and benchmark timings: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
- [abstract] Platform modules the formatted tables feed into: "a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
- [other] Repository availability: "CNIC-Proteomics/TurboPutative-web"
