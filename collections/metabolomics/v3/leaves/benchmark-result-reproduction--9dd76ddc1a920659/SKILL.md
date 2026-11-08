---
name: benchmark-result-reproduction
description: Use when a published software benchmark reports per-dataset success/failure
  counts or percentages (e.g., '38/41 correct outputs, 92.7%') and the goal is to
  verify them by re-executing the pipeline on the same study IDs.
when_to_use_negative:
- The goal is scientific re-analysis of the metabolomics data (e.g., classification),
  not verification of the conversion pipeline — re-analysis is infeasible when class
  metadata is unavailable in the repositories.
- The task is to extend or add data-processing algorithms to the tool — CloMet deliberately
  leaves processing out of scope.
- The input is the user's own private/local data routed via the LOCAL tab, which is
  outside the public-repository benchmark scope.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: CloMet
  role: Open-source modular platform whose LoadDataset pipeline is re-run per study
    ID to download raw/NMR data and convert it into the standard output format being
    benchmarked
  repo: github.com/rmallol/clomet
- name: Docker
  role: Container runtime used to execute the CloMet LoadDataset pipeline reproducibly
    for all 93 study runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:edf9cdc592a6163d7e76b78e2be141f895313d01fd7eb7605c0081f6ac579acc
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboLights
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# benchmark-result-reproduction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reproduces a paper's end-to-end success-rate benchmarks by re-running the tool's pipeline (here, CloMet's LoadDataset) on the same cohort of public repository studies and comparing aggregate correct-output counts against published figures (e.g., Figures 8B/8C of the CloMet paper).

## When to use

Use when a published software benchmark reports per-dataset success/failure counts or percentages (e.g., '38/41 correct outputs, 92.7%') and the goal is to verify them by re-executing the pipeline on the same study IDs.

## When NOT to use

- The goal is scientific re-analysis of the metabolomics data (e.g., classification), not verification of the conversion pipeline — re-analysis is infeasible when class metadata is unavailable in the repositories.
- The task is to extend or add data-processing algorithms to the tool — CloMet deliberately leaves processing out of scope.
- The input is the user's own private/local data routed via the LOCAL tab, which is outside the public-repository benchmark scope.

## Inputs

- List of study IDs for the benchmark cohort (e.g., 41 accepted data sets)
- List of study IDs for the comparison cohort (e.g., 52 studies from MetaboLights and Metabolomics Workbench)
- CloMet Docker container with the LoadDataset pipeline
- Published target benchmark values (e.g., 92.7% and 73.1% from Figures 8B/8C)

## Outputs

- Per-study correct/incorrect status table (benchmark_results.csv)
- Aggregate correct-output counts per cohort (e.g., 38/41 and 38/52)
- Success-rate percentages compared against the published figures

## How to apply

Retrieve the list of study IDs defining both benchmark cohorts (for the CloMet paper: 41 accepted data sets and 52 studies drawn from MetaboLights and Metabolomics Workbench). For each study ID, run the CloMet LoadDataset pipeline inside the provided Docker container to download the raw/NMR data and convert it to the analysis-ready format usable in MetaboAnalyst or Workflows4Metabolomics; the tool is designed so that 'inserting a study id' yields a ready-to-use data set with minimal interaction. For each run (93 total across both cohorts), inspect the converted output directory and judge whether the conversion produced a correct, ready-to-use data set rather than a partial or failed conversion. Aggregate correct-output counts separately per cohort and compute success percentages, writing per-study statuses and aggregate rates to benchmark_results.csv. Validate by checking the reproduced counts against the published rates — 38/41 (92.7%) and 38/52 (73.1%) in Figures 8B and 8C.

## Related tools

- **CloMet** (Open-source modular platform whose LoadDataset pipeline is re-run per study ID to download raw/NMR data and convert it into the standard output format being benchmarked) — github.com/rmallol/clomet
- **Docker** (Container runtime used to execute the CloMet LoadDataset pipeline reproducibly for all 93 study runs)

## Examples

```
docker run --rm -v $(pwd):/data rmallol/clomet LoadDataset MTBLS1
```

## Evaluation signals

- Reproduced correct-output count for the accepted cohort equals 38 out of 41 (92.7%)
- Reproduced correct-output count for the repository-wide cohort equals 38 out of 52 (73.1%)
- benchmark_results.csv contains a per-study status entry for every run (41 accepted + 52 studies = 93 runs)
- Each study run either produces a correct, ready-to-use output data set or is explicitly marked as failed — no partial/unclassified outcomes remain
- Cohort membership matches the original lists of accepted data sets and MetaboLights/Metabolomics Workbench studies

## Limitations

- Success is judged only on producing a ready-to-use converted output; CloMet itself does not implement data processing algorithms, so downstream analysis quality is not covered by the benchmark.
- Re-analysis for classification problems is infeasible when class metadata is not available alongside the metabolomics data in public repositories.
- CloMet is focused on NMR-based untargeted metabolomics data from MetaboLights and Metabolomics Workbench; its maintenance and extension depend on the open-source developer community, so pipeline behavior may drift from the published benchmark over time.

## Evidence

- [abstract] converts raw and NMR-based metabolomics data from MetaboLights and Metabolomics Workbench: "converts raw and NMR-based metabolomics data from MetaboLights and Metabolomics Workbench"
- [abstract] validated CloMet and output data on repository datasets: "We validated both CloMet and the output data using data sets from these repositories."
- [discussion] study-id retrieval yields ready-to-use dataset: "by just inserting a study id our users can get a ready-to-use data set with the minimal interaction possible"
