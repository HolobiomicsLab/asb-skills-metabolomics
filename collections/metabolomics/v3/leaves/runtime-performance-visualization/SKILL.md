---
name: runtime-performance-visualization
description: 'Use when comparing normalization or data-processing methods for high-throughput
  feasibility: runtimes must be measured under a parallelization sweep (1–6 cores)
  and rendered as compounds-per-minute scaling curves on targeted and untargeted metabolomics
  data.'
when_to_use_negative:
- When the goal is normalization quality (drift/batch-effect reduction, replicate
  concordance) rather than computational throughput — use the article's performance
  benchmarking instead.
- When the method under test cannot be parallelized or is only available behind an
  interactive interface (e.g. SERFF's Shiny web interface), where per-core scaling
  is not applicable — report wall-clock timing only.
- When data are not metabolomics feature/run matrices at comparable scale; the compounds-per-minute
  metric and the BioHEART/FAME reference timings do not transfer to unrelated domains.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Metanorm
  role: Provides the QC-RLSC, QC-RSC, rLOESS, rGAM, and tGAM normalization methods
    whose runtimes are timed across 1–6 cores; exploits pleasingly parallel processing
    per compound.
  repo: https://github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: R implementation installed from GitHub and called with model/type/QCcheck/batch
    arguments to execute each timed normalization run.
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment (version >= 4.4.0 required) in which Metanorm and TIGER
    are executed and timings are collected across the core sweep.
- name: TIGER
  role: Multivariate comparison method run through its R package under the same dataset/core
    scenarios; benchmarked at ~64 s on BioHEART and ~115 min on FAME.
- name: SERFF
  role: Multivariate comparison method benchmarked through its Shiny web interface;
    core scaling is not applicable for this tool.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:2746f135daf6b329cfc43b666eeef5e5b60d36285593bee6c99f30dcab2eeed2
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# runtime-performance-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the computational runtimes and parallel-scaling behavior of metabolomics normalization methods (QC-RSC, QC-RLSC, rLOESS, rGAM, tGAM, TIGER, SERFF) by expressing throughput as normalized compounds per minute across 1–6 compute cores on targeted and untargeted datasets, then plotting scaling curves to expose tapering parallel gains.

## When to use

Use when comparing normalization or data-processing methods for high-throughput feasibility: runtimes must be measured under a parallelization sweep (1–6 cores) and rendered as compounds-per-minute scaling curves on targeted and untargeted metabolomics data.

## When NOT to use

- When the goal is normalization quality (drift/batch-effect reduction, replicate concordance) rather than computational throughput — use the article's performance benchmarking instead.
- When the method under test cannot be parallelized or is only available behind an interactive interface (e.g. SERFF's Shiny web interface), where per-core scaling is not applicable — report wall-clock timing only.
- When data are not metabolomics feature/run matrices at comparable scale; the compounds-per-minute metric and the BioHEART/FAME reference timings do not transfer to unrelated domains.

## Inputs

- BioHEART targeted metabolomics dataset (53 compounds, 1361 sample runs, 15 batches)
- FAME untargeted metabolomics dataset (8913 features, 618 runs, 15 batches; 9 batches reported)
- Metanorm R package input objects: rawdata numeric matrix (rows = compounds, columns = samples), batch vector per sample run, and QC/sample type vector (metanorm.qc)
- Core-count sweep specification (1 to 6 compute cores)

## Outputs

- Method-by-dataset-by-core-count runtime table expressed as normalized compounds per minute
- Compounds-per-minute scaling curves versus number of cores for each method and dataset
- Timing comparison against reported values (e.g. tGAM ~136 s and TIGER 64 s on BioHEART; tGAM ~15 min and TIGER ~115 min on FAME)

## How to apply

Obtain the two reference scenarios — the BioHEART targeted dataset (53 compounds, 1361 sample runs, 15 batches) and the FAME untargeted dataset (8913 features, 618 runs, 9 batches) — and install the Metanorm R package from GitHub. Run each Metanorm method (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) with default arguments except rGAM/tGAM, which are run with the QCcheck argument active, timing each analysis from 1 to 6 compute cores on both datasets; insert a 2-minute break between consecutive method analyses to prevent CPU throttling. Run TIGER through its R package under identical scenarios, and run SERFF through its Shiny web interface (core scaling not applicable). Express every runtime as normalized compounds per minute, compile a method-by-dataset-by-core-count table, and plot compounds-per-minute scaling curves versus core count per method and dataset. Expect tapering/declining curves for the fastest methods on the small BioHEART set versus steady improvement with more cores on FAME; QC-RSC should emerge fastest overall while tGAM (~136 s on BioHEART) and TIGER (64 s) are the slowest tier, and rGAM (~18 s) is a faster robust alternative.

## Related tools

- **Metanorm** (Provides the QC-RLSC, QC-RSC, rLOESS, rGAM, and tGAM normalization methods whose runtimes are timed across 1–6 cores; exploits pleasingly parallel processing per compound.) — https://github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (R implementation installed from GitHub and called with model/type/QCcheck/batch arguments to execute each timed normalization run.) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment (version >= 4.4.0 required) in which Metanorm and TIGER are executed and timings are collected across the core sweep.)
- **TIGER** (Multivariate comparison method run through its R package under the same dataset/core scenarios; benchmarked at ~64 s on BioHEART and ~115 min on FAME.)
- **SERFF** (Multivariate comparison method benchmarked through its Shiny web interface; core scaling is not applicable for this tool.)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch)
```

## Evaluation signals

- Reproduced timings match reported values: tGAM ~136 s and TIGER 64 s on BioHEART; tGAM ~15 min and TIGER ~115 min on FAME.
- QC-RSC is the fastest method overall across both datasets and the core sweep.
- Scaling curves for the fastest methods taper or decline with more cores on the small BioHEART set (53 compounds), while FAME (8913 features) shows steady improvement up to 6 cores.
- A 2-minute break between consecutive method analyses was inserted, so timings show no CPU-throttling drift across the method sequence.
- rGAM runtime (~18 s on BioHEART) is dramatically lower than tGAM (~136 s), consistent with the README's statement that rGAM and rLOESS are faster than tGAM.

## Limitations

- SERFF is accessed only via a Shiny web interface, so its core-scaling behavior cannot be measured in the sweep.
- The small BioHEART dataset (53 compounds) underutilizes available cores, producing tapering or declining scaling curves that should not be extrapolated to larger datasets.
- The article flags that benchmarking must remain neutral — care should be taken not to potentially favor any of the benchmarked methods (e.g. via argument tuning).
- The package has been extensively tested, but unforeseen circumstances may cause unexpected behaviour; issues should be raised on GitHub.
- TIGER and SERFF are not benchmarked inside the article's own normalization evaluation; this throughput comparison addresses that gap externally.

## Evidence
