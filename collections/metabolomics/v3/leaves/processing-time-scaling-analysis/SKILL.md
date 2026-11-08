---
name: processing-time-scaling-analysis
description: Use when evaluating whether TurboPutative's RESTful API workflow can
  handle a planned metabolomics annotation dataset, or when comparing end-to-end runtime
  scaling of a web-based annotation-processing pipeline across input sizes.
when_to_use_negative:
- The goal is annotation accuracy or entry-reduction quality (the reported 80%–90%
  putative-annotation reduction) rather than runtime performance
- The input is not a tabular compound-name/mass table accepted by TurboPutative's
  web services (e.g., raw spectra or mzML files)
- Interactive, single-user GUI processing of a small table where runtime scaling is
  irrelevant
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TurboPutative
  role: Web-based platform whose four modules (Tagger, REname, RowMerger, TPMetrics)
    are exercised end-to-end via its RESTful API services for the timing benchmarks
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:5e4ba919f1b295897d0ee405ef937b01985628d7e1e082873d9d52ec299982e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# processing-time-scaling-analysis

## Summary

Benchmark TurboPutative's end-to-end processing time as a function of TSV input table size by submitting compound-name/mass tables of varying row counts to its RESTful API and recording wall-clock times, verifying near-linear scaling across the 500–30,000-row range.

## When to use

Use when evaluating whether TurboPutative's RESTful API workflow can handle a planned metabolomics annotation dataset, or when comparing end-to-end runtime scaling of a web-based annotation-processing pipeline across input sizes.

## When NOT to use

- The goal is annotation accuracy or entry-reduction quality (the reported 80%–90% putative-annotation reduction) rather than runtime performance
- The input is not a tabular compound-name/mass table accepted by TurboPutative's web services (e.g., raw spectra or mzML files)
- Interactive, single-user GUI processing of a small table where runtime scaling is irrelevant

## Inputs

- TSV table with 500 compound-name/mass rows in TurboPutative's accepted input format
- TSV table with 30,000 compound-name/mass rows, structurally identical to the small table
- TurboPutative RESTful API endpoint (https://proteomics.cnic.es/TurboPutative/)

## Outputs

- api_benchmark_timings.csv with recorded wall-clock processing times for both table sizes
- Scaling assessment confirming near-linear runtime growth across the 500–30,000-row range

## How to apply

Build two structurally identical TSV input tables in the format accepted by TurboPutative's web services — one with 500 compound-name/mass rows and one with 30,000 rows. Submit each table to the TurboPutative RESTful API endpoint via HTTP POST and measure wall-clock time from submission to completion of the automated Tagger/REname/RowMerger/TPMetrics workflow. Record the two processing times (reported as approximately 10 s for 500 rows and approximately 250 s for 30,000 rows) to a CSV such as api_benchmark_timings.csv. Validate the measurements against near-linear scaling by checking that runtime grows roughly proportionally with row count across the tested range, i.e., ~60× rows yields ~25× time in the reported benchmarks; large deviations indicate non-linear bottleneck behavior worth investigating.

## Related tools

- **TurboPutative** (Web-based platform whose four modules (Tagger, REname, RowMerger, TPMetrics) are exercised end-to-end via its RESTful API services for the timing benchmarks) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Measured processing time for the 500-row table is approximately 10 seconds
- Measured processing time for the 30,000-row table is approximately 250 seconds
- Ratio of measured times to row counts is consistent with near-linear scaling in the tested range (no super-linear blow-up)
- api_benchmark_timings.csv contains recorded wall-clock times for both table sizes with matching input schemas
- Both submissions return complete TurboPutative workflow outputs (not partial/module-only results) so timings reflect true end-to-end processing

## Limitations

- Near-linear scaling is validated only within the tested 500–30,000-row range; extrapolation beyond 30,000 rows is unsupported
- The published discussion/changelog content is absent from the indexed source, so additional runtime caveats from the authors are unavailable
- Timing benchmarks assume the workflow is run via RESTful API services; interactive web-GUI use may exhibit different overhead

## Evidence

- [abstract] TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
- [abstract] flexible and user-friendly web-based platform composed of four modules: "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
- [other] CNIC-Proteomics/TurboPutative-web: "CNIC-Proteomics/TurboPutative-web"
