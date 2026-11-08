---
name: metabolomics-normalization-benchmark-execution
description: Use when comparing computational throughput of metabolomics normalization
  methods across parallel core counts, or when estimating how long tGAM/rGAM/rLOESS/TIGER/SERFF
  will take on a targeted or untargeted dataset.
when_to_use_negative:
- Evaluating normalization quality (drift removal, batch-effect reduction, differential-abundance
  recovery) rather than computational throughput — this skill only measures speed/scaling.
- Benchmarking SERFF's parallel scaling — it is accessed through a Shiny web interface,
  so core-count scaling cannot be controlled or measured for it.
- Datasets far outside the BioHEART/FAME size range without re-deriving expected runtime
  envelopes; the known absolute timings (e.g., tGAM 136 s on 53 compounds, ~15 min
  on 8913 features) will not transfer directly.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Metanorm
  role: R package that implements QC-RLSC, QC-RSC, rLOESS, rGAM and tGAM with parallel
    (pleasingly parallel) processing for the benchmark runs
  repo: https://github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Provides the metanorm() normalization call whose per-method runtime is measured
    under 1–6 core configurations
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version ≥ 4.4.0) required by the Metanorm and TIGER packages
    used for the throughput benchmark
- name: TIGER
  role: Multivariate normalization baseline run through its R package under the same
    core-scaling scenarios to compare runtime against Metanorm methods
- name: SERFF
  role: Multivariate normalization baseline accessed via its Shiny web interface;
    benchmarked without core-scaling since core counts are not controllable there
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:c8ff76b052fdca043c861001b4eb63e798e343157229e51a4576b0f9a85d9fa2
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-normalization-benchmark-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes benchmark runs of Metanorm's normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) and the multivariate baselines TIGER and SERFF, timing each on targeted (BioHEART) and untargeted (FAME) metabolomics datasets across 1–6 compute cores and expressing runtimes as normalized compounds per minute.

## When to use

Use when comparing computational throughput of metabolomics normalization methods across parallel core counts, or when estimating how long tGAM/rGAM/rLOESS/TIGER/SERFF will take on a targeted or untargeted dataset.

## When NOT to use

- Evaluating normalization quality (drift removal, batch-effect reduction, differential-abundance recovery) rather than computational throughput — this skill only measures speed/scaling.
- Benchmarking SERFF's parallel scaling — it is accessed through a Shiny web interface, so core-count scaling cannot be controlled or measured for it.
- Datasets far outside the BioHEART/FAME size range without re-deriving expected runtime envelopes; the known absolute timings (e.g., tGAM 136 s on 53 compounds, ~15 min on 8913 features) will not transfer directly.

## Inputs

- Untargeted metabolomics intensity matrix (e.g., FAME: 8913 features × 618 sample runs, 9 batches)
- Targeted metabolomics intensity matrix (e.g., BioHEART: 53 compounds × 1361 sample runs, 15 batches)
- Sample-type vector distinguishing QC runs from biological samples (metanorm.qc)
- Batch assignment vector per sample run
- Compute environment with configurable core counts 1–6
- Metanorm R package (UGent-LIMET/Metanorm) and TIGER R package
- SERFF Shiny web interface

## Outputs

- Method × dataset × core-count runtime table expressed as normalized compounds per minute
- Compounds-per-minute scaling curves versus number of cores for each method and dataset
- Reproducible benchmark timings validating that tGAM ~136 s and TIGER ~64 s on BioHEART, and tGAM ~15 min and TIGER ~115 min on FAME
- Evidence of which method is fastest overall (QC-RSC) and how throughput scales with parallelism per dataset scale

## How to apply

Install the Metanorm R package (R ≥ 4.4.0) and TIGER's R package; access SERFF via its Shiny web interface. Run each Metanorm method (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) with default arguments except rGAM and tGAM, which must be run with the QCcheck argument active, timing each analysis with 1–6 cores on both the targeted (BioHEART: 53 compounds, 1361 sample runs, 15 batches) and untargeted (FAME: 8913 features, 618 runs, 9 batches) scenarios. Insert a 2-minute break between consecutive method analyses to prevent CPU throttling — otherwise timings will be confounded by thermal effects. Convert every runtime to normalized compounds per minute (compounds analyzed divided by wall-clock seconds), then compile a method × dataset × core-count table and plot compounds-per-minute scaling curves versus cores. Expect tapering or declining scaling curves for the fast methods (QC-RSC, rGAM, rLOESS) on the small BioHEART set versus steady improvement with more cores on FAME. Core scaling is not applicable for SERFF since it is run through a web interface; the rationales are that compounds-per-minute normalizes runtime across datasets of different sizes and that the 2-min break guards against throttling artifacts.

## Related tools

- **Metanorm** (R package that implements QC-RLSC, QC-RSC, rLOESS, rGAM and tGAM with parallel (pleasingly parallel) processing for the benchmark runs) — https://github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Provides the metanorm() normalization call whose per-method runtime is measured under 1–6 core configurations) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version ≥ 4.4.0) required by the Metanorm and TIGER packages used for the throughput benchmark)
- **TIGER** (Multivariate normalization baseline run through its R package under the same core-scaling scenarios to compare runtime against Metanorm methods)
- **SERFF** (Multivariate normalization baseline accessed via its Shiny web interface; benchmarked without core-scaling since core counts are not controllable there)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Reproduced tGAM runtime on BioHEART lands near ~136 s with 1 core (TIGER near ~64 s; rGAM ~18 s).
- Reproduced tGAM runtime on FAME lands near ~15 min with TIGER near ~115 min.
- QC-RSC emerges as the fastest method overall in the compounds-per-minute table.
- Throughput scaling curves for the fast methods (QC-RSC, rGAM, rLOESS) show tapering or declining gains on the small BioHEART set, versus continued improvement with more cores on the FAME dataset.
- A 2-minute break between consecutive method analyses is enforced so no CPU-throttling artifact appears as a systematic timing drop.

## Limitations

- Absolute timings are dataset-specific: compounds-per-minute values from BioHEART (53 compounds) do not transfer to FAME (8913 features) and vice versa.
- SERFF benchmarking is limited because it runs through a Shiny web interface, precluding core-count scaling.
- CPU throttling between consecutive runs can distort timings unless the prescribed 2-minute break is respected.
- tGAM is the slowest Metanorm method (~136 s BioHEART, ~15 min FAME); the README notes rGAM and rLOESS are faster alternatives when runtime matters more than robustness.
- Benchmarking should avoid favoring any method; the article cautions that 'care should be taken to ensure neutrality, i.e., not potentially favoring any of the benchmarked methods'.

## Evidence

- [readme] Workflow prescription from the README: [evidence span withheld — anchor status: external_doc]
