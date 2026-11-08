---
name: api-latency-benchmarking
description: Use when you need to quantify how a TurboPutative-based pipeline's runtime
  scales with input table size, by submitting compound annotation TSV tables of different
  row counts to its RESTful API and measuring submission-to-completion times.
when_to_use_negative:
- Your goal is annotation reduction quality (e.g. the 80%-90% entry reduction from
  REname/RowMerger/TPMetrics), not runtime characterization — use the TurboPutative
  modules directly instead of benchmarking latency.
- Your input tables fall outside the tested 500–30,000-row range and you intend to
  extrapolate the reported scaling quantitatively.
- Your data is raw mass-spectrometry output rather than the compound-name/mass TSV
  format TurboPutative's web services accept.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web platform whose RESTful API services are submitted the benchmark TSV tables
    and timed end-to-end
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:e25c73c2c33d96ce0f2cd21726a052eccb9715ff9173f0f2bb720be5a8a4069d
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
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# api-latency-benchmarking

## Summary

Benchmark the end-to-end wall-clock processing time of TurboPutative's RESTful API services on TSV compound-name/mass tables, demonstrating near-linear scaling (~10 s for 500 rows, ~250 s for 30,000 rows).

## When to use

Use when you need to quantify how a TurboPutative-based pipeline's runtime scales with input table size, by submitting compound annotation TSV tables of different row counts to its RESTful API and measuring submission-to-completion times.

## When NOT to use

- Your goal is annotation reduction quality (e.g. the 80%-90% entry reduction from REname/RowMerger/TPMetrics), not runtime characterization — use the TurboPutative modules directly instead of benchmarking latency.
- Your input tables fall outside the tested 500–30,000-row range and you intend to extrapolate the reported scaling quantitatively.
- Your data is raw mass-spectrometry output rather than the compound-name/mass TSV format TurboPutative's web services accept.

## Inputs

- TSV table with 500 compound-name/mass rows conforming to TurboPutative's accepted input format
- Structurally identical TSV table with 30,000 compound-name/mass rows
- TurboPutative RESTful API endpoint (https://proteomics.cnic.es/TurboPutative/)

## Outputs

- api_benchmark_timings.csv containing the measured end-to-end processing times for the 500-row and 30,000-row tables

## How to apply

Build two structurally identical TSV input tables in the format accepted by TurboPutative's web services — one with 500 compound-name/mass rows and one with 30,000 rows. Submit each table to the TurboPutative RESTful API endpoint (https://proteomics.cnic.es/TurboPutative/) via HTTP POST, timing the wall-clock duration from submission to completion. Record both timings to api_benchmark_timings.csv. Validate against the reported near-linear scaling behavior: the 30,000-row table is ~60x larger than the 500-row table yet takes only ~25x longer (~250 s vs ~10 s), reflecting sub-linear overhead in the tested 500–30,000-row range. Rationale: TurboPutative's automated workflow is explicitly designed for inclusion in programmed bioinformatics pipelines through its RESTful API services, so end-to-end latency through that API is the deployment-relevant performance metric.

## Related tools

- **TurboPutative** (Web platform whose RESTful API services are submitted the benchmark TSV tables and timed end-to-end) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Examples

```
python -c "import requests,time; t0=time.time(); requests.post('https://proteomics.cnic.es/TurboPutative/',files={'file':open('table_500rows.tsv','rb')}); print(time.time()-t0)"
```

## Evaluation signals

- Measured time for the 500-row table is approximately 10 s.
- Measured time for the 30,000-row table is approximately 250 s.
- Timing ratio between the two tables (~25x) is consistent with near-linear scaling across the 60x row-count increase.
- api_benchmark_timings.csv exists and contains one recorded wall-clock time per tested table.
- Both submitted TSV tables are structurally identical except for row count, isolating size as the only variable.

## Limitations

- Scaling behavior was validated only within the 500–30,000-row range; extrapolation beyond it is unsupported.
- Wall-clock timings reflect the hosted service at https://proteomics.cnic.es/TurboPutative/ and may vary with server load and network conditions.
- Timings cover the full four-module pipeline (Tagger, REname, RowMerger, TPMetrics) end-to-end, so per-module latency cannot be attributed from this benchmark alone.

## Evidence

- [abstract] TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
- [abstract] a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics): "TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
