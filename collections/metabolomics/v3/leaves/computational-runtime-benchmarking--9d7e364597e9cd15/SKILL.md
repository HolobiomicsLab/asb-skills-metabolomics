---
name: computational-runtime-benchmarking
description: Use when re-running a published computational pipeline (e.g. compareMS2
  pairwise distance calculations over PXD034932 serum datasets) and you need a measured
  runtime to compare against the paper's reported wall-clock time on its reference
  hardware.
when_to_use_negative:
- The goal is accuracy or biological validation of the tool's output (e.g. reconstructing
  the correct phylogenetic tree) rather than measuring computational cost — benchmarking
  says nothing about result quality.
- You only need a single pairwise comparison or a small exploratory subset rather
  than the full N-choose-2 batch; a partial run does not reproduce the published aggregate
  runtime.
- The workload is interactive GUI usage (compareMS2 GUI with real-time UPGMA tree
  updates) rather than the deterministic CLI batch the paper timed.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: tool whose 10 pairwise distance comparisons over the five serum datasets are
    the timed workload, run in command-line mode with default parameters
  repo: https://github.com/524D/compareMS2.git
- name: compareMS2 2.0
  role: the specific major version under benchmark, providing CLI and GUI modes and
    a default distance metric
  repo: https://github.com/524D/compareMS2.git
- name: Intel Xeon W-2135
  role: reference CPU (3.70 GHz) on which the paper's 40 min runtime was measured;
    hardware context for interpreting measured times
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:557bedfec8a7ca102c7b25ed7ac7ce6f6d5bccf6e8fcfc0f212a95fdfa37fb24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - Intel Xeon W-2135
  - compareMS2
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# computational-runtime-benchmarking

## Summary

Measure wall-clock runtime of a scientific tool's core computation (here, the 10 pairwise tandem mass spectrum dataset comparisons in compareMS2) as a benchmark baseline, using a timing harness around the default-parameter run and reporting the hardware used.

## When to use

Use when re-running a published computational pipeline (e.g. compareMS2 pairwise distance calculations over PXD034932 serum datasets) and you need a measured runtime to compare against the paper's reported wall-clock time on its reference hardware.

## When NOT to use

- The goal is accuracy or biological validation of the tool's output (e.g. reconstructing the correct phylogenetic tree) rather than measuring computational cost — benchmarking says nothing about result quality.
- You only need a single pairwise comparison or a small exploratory subset rather than the full N-choose-2 batch; a partial run does not reproduce the published aggregate runtime.
- The workload is interactive GUI usage (compareMS2 GUI with real-time UPGMA tree updates) rather than the deterministic CLI batch the paper timed.

## Inputs

- five serum LC-MS/MS datasets (each containing 42,629 to 47,626 tandem mass spectra) from ProteomeXchange accession PXD034932
- installed compareMS2 2.0 (CLI mode, default parameters)
- timing harness that records wall-clock start/stop times for the batch of pairwise comparisons

## Outputs

- measured wall-clock runtime for the complete batch of 10 pairwise distance comparisons
- comparems2_runtimes.csv with per-run and total wall-clock times and the CPU model/clock speed used
- comparison against the published baseline (40 min with default parameters on Intel Xeon W-2135, 3.70 GHz)

## How to apply

Identify the tool's core computational workload from the paper — for compareMS2 2.0 this is the pairwise comparison of all LC-MS/MS dataset pairs (N=5 datasets gives 10 pairwise comparisons of sets of 42,629–47,626 tandem mass spectra each). Acquire the same input data the paper used, here the five serum datasets from ProteomeXchange accession PXD034932. Install the tool as the paper specifies: clone the 524D/compareMS2 repository, enable corepack, and run yarn (NodeJS 22.x per README), or use the compareMS2 2.0 CLI in command-line mode. Run the workload with the paper's parameters (default parameters) and wrap the execution in a timing harness that records wall-clock start and stop times covering the full batch of comparisons. Record the CPU model and clock speed alongside, because the reference result is hardware-dependent: the paper reports 40 min for the 10 pairwise comparisons on an Intel Xeon W-2135 at 3.70 GHz. Write per-run and total times to a CSV (e.g. comparems2_runtimes.csv) and judge success by same-order-of-magnitude agreement with the reported value, not exact equality, since compute hardware differs.

## Related tools

- **compareMS2** (tool whose 10 pairwise distance comparisons over the five serum datasets are the timed workload, run in command-line mode with default parameters) — https://github.com/524D/compareMS2.git
- **compareMS2 2.0** (the specific major version under benchmark, providing CLI and GUI modes and a default distance metric) — https://github.com/524D/compareMS2.git
- **Intel Xeon W-2135** (reference CPU (3.70 GHz) on which the paper's 40 min runtime was measured; hardware context for interpreting measured times)

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn
```

## Evaluation signals

- Number of pairwise comparisons matches N datasets choose 2: five datasets must yield exactly 10 pairwise distance computations, matching the paper's comparison count.
- All five input datasets are used and their tandem mass spectra counts fall within the published 42,629–47,626 range per dataset.
- Run parameters equal the paper's default parameters; any deviation must be recorded and excludes direct comparison to the 40 min baseline.
- Measured total wall-clock time is same order of magnitude as the reported 40 min when CPU class is comparable to an Intel Xeon W-2135 at 3.70 GHz.
- comparems2_runtimes.csv exists with per-run and total wall-clock times plus CPU model and clock speed recorded.

## Limitations

- Runtime is strongly hardware-dependent: the 40 min figure was measured on an Intel Xeon W-2135 at 3.70 GHz, so acceptance is same-order-of-magnitude rather than exact match across machines.
- compareMS2 performs N(N-1)/2 pairwise comparisons — 100 datasets require 4950 comparisons taking several hours — so scaling limits runtime benchmarking to modest dataset counts.
- Default parameters must be reproduced exactly for the benchmark to be comparable; the metric is sensitive to m/z windows and spectral similarity threshold settings.
- Installation is platform- and NodeJS-version-sensitive: due to an electron/forge bug the currently latest NodeJS version cannot be used (README, June 2026), and Windows corepack enable requires administrator rights.

## Evidence

- [results] reported runtime baseline: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [results] reference hardware: "on a PC with an Intel Xeon W-2135 CPU running at 3.70 GHz"
- [results] input dataset scale: "each containing between 42,629 and 47,626 tandem mass spectra"
- [readme] installation steps: "git clone https://github.com/524D/compareMS2
cd compareMS2
yarn"
- [results] scaling cost of the workload: "comparing 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours"
