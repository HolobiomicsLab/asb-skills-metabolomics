---
name: restful-api-workflow-automation
description: Use when an untargeted metabolomics putative-annotation table (e.g.,
  compound-name/mass TSV) must be processed headlessly — via TurboPutative's RESTful
  API — inside an automated pipeline, or when benchmarking runtime scaling with input
  row count.
when_to_use_negative:
- Interactive, manual exploration of a single small annotation table where the web
  UI suffices and scripting overhead is unnecessary
- Analyses requiring metabolite identification from raw spectra (e.g., mzML files)
  rather than handling of pre-computed putative annotation tables
- Inputs outside the annotation-table format TurboPutative's web services accept (the
  tested range is 500–30,000 rows)
tools:
- name: TurboPutative
  role: Web platform whose four modules (Tagger, REname, RowMerger, TPMetrics) are
    invoked programmatically through its RESTful API services to process annotation
    tables end-to-end
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:6e3b6f458bd9430b8750bd79742ae358ecae316b7d056d064892c2ed023933e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# restful-api-workflow-automation

## Summary

Automates TurboPutative's end-to-end metabolite-annotation workflow (Tagger, REname, RowMerger, TPMetrics) by submitting input tables programmatically through its RESTful API services, enabling inclusion in scripted bioinformatics pipelines and wall-clock benchmarking of processing time versus input table size.

## When to use

Use when an untargeted metabolomics putative-annotation table (e.g., compound-name/mass TSV) must be processed headlessly — via TurboPutative's RESTful API — inside an automated pipeline, or when benchmarking runtime scaling with input row count.

## When NOT to use

- Interactive, manual exploration of a single small annotation table where the web UI suffices and scripting overhead is unnecessary
- Analyses requiring metabolite identification from raw spectra (e.g., mzML files) rather than handling of pre-computed putative annotation tables
- Inputs outside the annotation-table format TurboPutative's web services accept (the tested range is 500–30,000 rows)

## Inputs

- TSV input table of compound-name/mass rows conforming to TurboPutative's accepted input format (500–30,000 rows tested)
- HTTP POST request payload submitted to the TurboPutative RESTful API endpoint

## Outputs

- Processed TurboPutative output spanning the four modules (Tagger classification, REname annotation cleanup, RowMerger merged rows, TPMetrics scores)
- Measured wall-clock processing times per table size (e.g., api_benchmark_timings.csv)

## How to apply

Construct input tables in the format accepted by TurboPutative's web services (compound-name/mass rows, e.g., TSV), sized according to the benchmark ranges tested in the paper (500 to 30,000 rows). Submit each table to the TurboPutative RESTful API endpoint at https://proteomics.cnic.es/TurboPutative/ via HTTP POST and measure wall-clock time from submission to completion. The platform's four modules — Tagger (compound classification), REname (isomer/equivalent-compound recognition and redundant-data removal), RowMerger (dataset-size reduction), and TPMetrics (score based on adduct probability and feature correlations) — run as the automated, customizable workflow behind the service. Recorded timings should follow near-linear scaling: approximately 10 s for a 500-entry table and approximately 250 s for a 30,000-entry table; write measured values (e.g., api_benchmark_timings.csv) and validate them against these reported figures. Expect massive datasets to have their putative-annotation entries reduced by roughly 80%-90%, facilitating metabolite prioritization for pathway analysis.

## Related tools

- **TurboPutative** (Web platform whose four modules (Tagger, REname, RowMerger, TPMetrics) are invoked programmatically through its RESTful API services to process annotation tables end-to-end) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Measured end-to-end time for a 500-row input table is approximately 10 seconds
- Measured end-to-end time for a 30,000-row input table is approximately 250 seconds
- Timing ratio between large and small tables is consistent with near-linear runtime scaling (~25x time for ~60x rows, per reported figures)
- API submission-to-completion completes without error for both the 500-row and 30,000-row TSVs
- Benchmark timings for both sizes are recorded to api_benchmark_timings.csv

## Limitations

- Reported timings (~10 s for 500 rows, ~250 s for 30,000 rows) were established only within the tested 500–30,000-row range; scaling outside this range is not characterized
- The article's source document contains no discussion-section text or changelog, so no additional caveats on API stability, rate limits, or payload limits are documented
- Automation depends on the availability of the hosted service at https://proteomics.cnic.es/TurboPutative/

## Evidence

- [abstract] abstract: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
