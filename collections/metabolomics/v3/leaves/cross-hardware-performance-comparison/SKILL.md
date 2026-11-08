---
name: cross-hardware-performance-comparison
description: Use when a published method reports a concrete runtime on named hardware
  (e.g. compareMS2 2.0's 40 min for 10 pairwise serum dataset comparisons on an Intel
  Xeon W-2135 at 3.70 GHz) and a re-run's performance must be assessed on possibly
  different machines.
when_to_use_negative:
- The published article reports no runtime or hardware configuration for the step
  being reproduced, so there is no qualified baseline to compare against.
- The workload differs from the article's benchmark inputs (e.g. datasets with far
  fewer or more tandem mass spectra than the 42,629–47,626 range), making the 40 min
  baseline inapplicable.
- The goal is algorithmic correctness validation rather than performance assessment
  — use the article's phylogenetic tree reconstruction in compareMS2 and MEGA11 for
  that instead.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0121
tools:
- name: compareMS2 2.0
  role: 'Software under benchmarking: run in command-line mode with default parameters
    over the five serum datasets to perform the 10 pairwise distance comparisons being
    timed'
  repo: https://github.com/524D/compareMS2.git
- name: Intel Xeon W-2135
  role: Reference CPU of the article's reported baseline (40 min at 3.70 GHz); hardware
    of the re-run machine must be recorded and compared against it
- name: NodeJS
  role: Runtime dependency (version 22.22.3 per README) needed to install and execute
    compareMS2 during the benchmark run
  repo: https://nodejs.org/en/download/archive/v22.22.3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:5101f1a4d8aeaa95a6625b7d73c5e0bcaba17a951f3a9a08a98dee59363dfbc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - Intel Xeon W-2135
  - NodeJS
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-hardware-performance-comparison

## Summary

Benchmark a compute step's wall-clock runtime against the hardware-qualified baseline reported in the source article, here the 40 min compareMS2 2.0 runtime for 10 pairwise LC-MS/MS dataset comparisons on an Intel Xeon W-2135 at 3.70 GHz.

## When to use

Use when a published method reports a concrete runtime on named hardware (e.g. compareMS2 2.0's 40 min for 10 pairwise serum dataset comparisons on an Intel Xeon W-2135 at 3.70 GHz) and a re-run's performance must be assessed on possibly different machines.

## When NOT to use

- The published article reports no runtime or hardware configuration for the step being reproduced, so there is no qualified baseline to compare against.
- The workload differs from the article's benchmark inputs (e.g. datasets with far fewer or more tandem mass spectra than the 42,629–47,626 range), making the 40 min baseline inapplicable.
- The goal is algorithmic correctness validation rather than performance assessment — use the article's phylogenetic tree reconstruction in compareMS2 and MEGA11 for that instead.

## Inputs

- Five serum LC-MS/MS datasets from ProteomeXchange accession PXD034932, each containing 42,629 to 47,626 tandem mass spectra
- compareMS2 2.0 installed from https://github.com/524D/compareMS2.git (NodeJS 22.22.3, corepack-enabled yarn)
- Hardware metadata: CPU model and clock speed of the benchmarking machine

## Outputs

- comparems2_runtimes.csv with per-run and total wall-clock times for the 10 pairwise comparisons
- Hardware-qualified runtime statement (measured time + CPU model/clock) comparable to the article's reported 40 min on Intel Xeon W-2135 at 3.70 GHz

## How to apply

First download the exact benchmark inputs the article used — the five PXD034932 serum LC-MS/MS datasets, each containing 42,629 to 47,626 tandem mass spectra — since runtime scales with spectral content. Install the pinned software version (compareMS2 2.0 from github.com/524D/compareMS2 via git clone, corepack enable, yarn) and run the workload with default parameters, wrapping execution in a timing harness that records wall-clock start/stop times for the 10 pairwise comparisons. Record the CPU model and clock speed alongside the measured time, because the article qualified its baseline ('took 40 min with default parameters') by the Intel Xeon W-2135 CPU running at 3.70 GHz. Write per-run and total times to a machine-readable artifact such as comparems2_runtimes.csv. Judge success by comparing the measured wall-clock time against the reported baseline: since hardware differs, acceptance is a same-order-of-magnitude runtime as judged by expert review rather than an exact numeric threshold.

## Related tools

- **compareMS2 2.0** (Software under benchmarking: run in command-line mode with default parameters over the five serum datasets to perform the 10 pairwise distance comparisons being timed) — https://github.com/524D/compareMS2.git
- **Intel Xeon W-2135** (Reference CPU of the article's reported baseline (40 min at 3.70 GHz); hardware of the re-run machine must be recorded and compared against it)
- **NodeJS** (Runtime dependency (version 22.22.3 per README) needed to install and execute compareMS2 during the benchmark run) — https://nodejs.org/en/download/archive/v22.22.3

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && time yarn start  # time the 10 pairwise comparisons on PXD034932 serum data, log CPU model/clock to comparems2_runtimes.csv
```

## Evaluation signals

- The five input datasets each fall within the article's stated spectral range of 42,629 to 47,626 tandem mass spectra, confirming the benchmark inputs match the paper's.
- Exactly 10 pairwise comparisons are executed and timed (the full pairwise set for 5 datasets), matching the article's workload.
- compareMS2 is run with default parameters and version 2.0, matching the parameterization the 40 min baseline was reported under.
- Total wall-clock time is recorded in comparems2_runtimes.csv and is same-order-of-magnitude as the reported 40 min, with any deviation explainable by CPU model/clock differences.
- Hardware metadata (CPU model and clock speed, e.g. Intel Xeon W-2135 at 3.70 GHz) is logged alongside the runtime so the comparison is hardware-qualified.

## Limitations

- Runtime is hardware-dependent: the article reports 40 min on an Intel Xeon W-2135 at 3.70 GHz, so measurements on other CPUs are only same-order-of-magnitude comparable, not threshold-matchable.
- The article's GUI allows executions to be paused or terminated mid-run, which would corrupt a wall-clock timing measurement if used; the benchmark must run to completion uninterrupted.
- Scalability is superlinear in dataset count: the article notes that comparing 100 LC-MS/MS datasets requires 4950 pairwise comparisons taking several hours, so the 5-dataset baseline cannot be extrapolated naively to larger collections.

## Evidence

- [results] Reported wall-clock runtime baseline (40 min, 10 pairwise comparisons, default parameters): "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [results] Hardware qualification of the reported baseline: "on a PC with an Intel Xeon W-2135 CPU running at 3.70 GHz"
- [results] Benchmark dataset identity and spectral content range: "each containing between 42,629 and 47,626 tandem mass spectra"
- [readme] Installation procedure for the benchmarked software version: "git clone https://github.com/524D/compareMS2
cd compareMS2
yarn"
- [results] Runtime scales with number of pairwise comparisons, limiting extrapolation from the 5-dataset baseline: "comparing 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours"
