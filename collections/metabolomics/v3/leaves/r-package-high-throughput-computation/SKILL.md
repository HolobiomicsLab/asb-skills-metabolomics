---
name: r-package-high-throughput-computation
description: 'Use when metabolomics normalization must run on large feature matrices
  (e.g. FAME: 8913 features, 618 runs, 9 batches) within practical wall-clock time,
  or when benchmarking method runtimes and parallel scaling across 1–6 compute cores.'
when_to_use_negative:
- Benchmarking core scaling for SERFF, which is only accessible through its Shiny
  web interface and does not support core-count scaling
- Small datasets where the fastest methods (e.g. QC-RSC, QC-RLSC) show tapering or
  declining scaling curves with additional cores — extra cores yield little throughput
  gain there
- Use cases where automated multivariate outlier detection or multiple testing correction
  for downstream inference is required — these are explicitly out of the package's
  scope
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Metanorm R package
  role: Provides the parallelized (pleasingly parallel) normalization implementations
    being executed and benchmarked for throughput
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >= 4.4.0) in which Metanorm is installed and
    called; core counts 1–6 are controlled within R
  repo: https://cloud.r-project.org/index.html
- name: TIGER
  role: Multivariate normalization comparator run through its R package under the
    same benchmark scenarios
- name: SERFF
  role: Multivariate normalization comparator run through its Shiny web interface
    (no core scaling applicable)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:1d38d17ab5d815792c62abb1d9311772d67255e1aa180b58308898333fed3c27
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  - SERFF
  - TIGER
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# R package high-throughput computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Leverages the pleasingly parallel, compound-wise structure of metabolomics normalization in the Metanorm R package (R >= 4.4.0) to normalize large feature matrices efficiently across compute cores, and to benchmark throughput (normalized compounds per minute) of Metanorm methods against TIGER and SERFF.

## When to use

Use when metabolomics normalization must run on large feature matrices (e.g. FAME: 8913 features, 618 runs, 9 batches) within practical wall-clock time, or when benchmarking method runtimes and parallel scaling across 1–6 compute cores.

## When NOT to use

- Benchmarking core scaling for SERFF, which is only accessible through its Shiny web interface and does not support core-count scaling
- Small datasets where the fastest methods (e.g. QC-RSC, QC-RLSC) show tapering or declining scaling curves with additional cores — extra cores yield little throughput gain there
- Use cases where automated multivariate outlier detection or multiple testing correction for downstream inference is required — these are explicitly out of the package's scope

## Inputs

- rawdata: numerical matrix of unnormalized intensities (rows = compounds, columns = sample runs)
- batch: character vector assigning each sample run to a batch
- metanorm.qc: vector of sample types ('QC' vs other) per sample run
- Targeted benchmark dataset (BioHEART: 53 compounds, 1361 sample runs, 15 batches)
- Untargeted benchmark dataset (FAME: 8913 features, 618 runs, 9 batches)

## Outputs

- Normalized data matrix of the same dimensions as the input
- Method-by-dataset-by-core-count table of runtimes as normalized compounds per minute
- Compounds-per-minute versus core-count scaling curves per method and dataset
- Diagnostic pre/post-normalization plots (PCA score plots and per-compound intensity-vs-order plots) written to plotdir

## How to apply

Install the Metanorm R package directly from GitHub via pak::pak('UGent-LIMET/Metanorm') under R >= 4.4.0. Call metanorm() on a rawdata numerical matrix (rows = compounds, columns = samples) together with a 'batch' vector and a 'metanorm.qc' sample-type vector, choosing model = 'tGAM' (default, recommended), 'rGAM', 'rLOESS', 'QC-RLSC', or 'QC-RSC'; the implementations capitalize on the problem's pleasingly parallel nature so per-compound fits distribute over 1–6 cores. For throughput benchmarking, run each method with default arguments except rGAM/tGAM with QCcheck active, time each analysis per core count on targeted (BioHEART: 53 compounds, 1361 sample runs, 15 batches) and untargeted (FAME) data, insert a 2 min break between consecutive method analyses to prevent CPU throttling, and express runtimes as normalized compounds per minute. Include TIGER (R package) and SERFF (Shiny web interface) as comparators; SERFF core scaling is not applicable. Validate timings against reported values (tGAM ~136 s and TIGER 64 s on BioHEART; tGAM ~15 min and TIGER ~115 min on FAME; QC-RSC fastest overall) and plot compounds-per-minute scaling curves per method and dataset.

## Related tools

- **Metanorm R package** (Provides the parallelized (pleasingly parallel) normalization implementations being executed and benchmarked for throughput) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >= 4.4.0) in which Metanorm is installed and called; core counts 1–6 are controlled within R) — https://cloud.r-project.org/index.html
- **TIGER** (Multivariate normalization comparator run through its R package under the same benchmark scenarios)
- **SERFF** (Multivariate normalization comparator run through its Shiny web interface (no core scaling applicable))

## Examples

```
pak::pak("UGent-LIMET/Metanorm"); normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Throughput table compiles every runtime as normalized compounds per minute for each method, dataset, and core count (1–6)
- Reproduced timings match reported values: tGAM ~136 s and TIGER 64 s on BioHEART; tGAM ~15 min and TIGER ~115 min on FAME
- QC-RSC emerges as the fastest method overall in the runtime comparison
- Scaling curves show tapering/declining gains with more cores for the fastest methods on the small BioHEART set, but steady improvement with more cores on FAME
- A 2 min break is inserted between consecutive method analyses to prevent CPU throttling from distorting timings

## Limitations

- Timings are sensitive to CPU throttling, requiring a 2 min break between consecutive method analyses during benchmarking
- Fastest methods show tapering or declining scaling on small datasets (BioHEART), so added cores do not improve throughput there
- Benchmarking neutrality is a concern — care should be taken to ensure results do not potentially favor any of the benchmarked methods
- Unexpected differences between QC samples and biological samples require due analyst scrutiny rather than automated handling

## Evidence

- [readme] R version requirement: [evidence span withheld — anchor status: external_doc]
- [readme] robustness of the package: [evidence span withheld — anchor status: external_doc]
