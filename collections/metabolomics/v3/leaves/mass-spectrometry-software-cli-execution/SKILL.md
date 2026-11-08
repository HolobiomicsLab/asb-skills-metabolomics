---
name: mass-spectrometry-software-cli-execution
description: Use when you need to reproduce or benchmark compareMS2's pairwise tandem
  mass spectrum comparisons from the CLI — e.g. timing the 10 pairwise comparisons
  of five serum datasets under default parameters — rather than using the GUI.
when_to_use_negative:
- When you need interactive visualization (phylogenetic tree, spectra2species bar
  chart, or heatmap) rather than the raw pairwise distance computation — use the compareMS2
  GUI for those modes.
- When the input is identified-peptide data or a database-search result rather than
  raw tandem mass spectrum sets; compareMS2 compares sets of tandem mass spectra directly,
  not peptide identifications.
- When you need exact runtime reproduction across different hardware — the paper's
  40 min figure is specific to an Intel Xeon W-2135 at 3.70 GHz and is hardware-dependent.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: 'The main tool: computes distances between sets of tandem mass spectra; run
    in command-line mode for benchmarking and batch execution over the pairwise comparisons.'
  repo: https://github.com/524D/compareMS2.git
- name: compareMS2 2.0
  role: The updated version under test, providing the default parameter set and the
    pairwise spectrum-set distance metric being timed.
  repo: https://github.com/524D/compareMS2
- name: Intel Xeon W-2135
  role: Reference CPU (3.70 GHz) on which the paper's 40 min baseline for 10 pairwise
    comparisons was measured.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:53455aa219cd1292e80dcfe7c00cd5025d706f70ce879ae031bdd4893de17e91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - Intel Xeon W-2135
  - compareMS2
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mass-spectrometry-software-cli-execution

## Summary

Runs compareMS2 2.0 in command-line mode over a set of LC-MS/MS datasets (e.g. the five serum PXD034932 files with 42,629-47,626 tandem mass spectra each) to compute the full set of pairwise spectrum-set distances, wrapped in a wall-clock timing harness to establish a reproducible runtime baseline on the user's own CPU.

## When to use

Use when you need to reproduce or benchmark compareMS2's pairwise tandem mass spectrum comparisons from the CLI — e.g. timing the 10 pairwise comparisons of five serum datasets under default parameters — rather than using the GUI.

## When NOT to use

- When you need interactive visualization (phylogenetic tree, spectra2species bar chart, or heatmap) rather than the raw pairwise distance computation — use the compareMS2 GUI for those modes.
- When the input is identified-peptide data or a database-search result rather than raw tandem mass spectrum sets; compareMS2 compares sets of tandem mass spectra directly, not peptide identifications.
- When you need exact runtime reproduction across different hardware — the paper's 40 min figure is specific to an Intel Xeon W-2135 at 3.70 GHz and is hardware-dependent.

## Inputs

- MGF (or equivalent tandem mass spectrum) files for each LC-MS/MS dataset to be compared
- compareMS2 2.0 installed from the 524D/compareMS2 repository
- Optional: a timing harness or shell wrapper that records wall-clock start/stop timestamps

## Outputs

- Pairwise distance/similarity matrix over the compared datasets (lower-triangular pairwise comparisons)
- Wall-clock runtime measurements for the batch of pairwise comparisons (e.g. comparems2_runtimes.csv)

## How to apply

Install compareMS2 2.0 from the 524D/compareMS2 GitHub repository (git clone, install NodeJS 22.22.3, corepack enable, yarn), then execute compareMS2 in command-line mode over the MGF files for all five LC-MS/MS datasets to perform the n(n-1)/2 pairwise comparisons of tandem mass spectrum sets — for five datasets that is 10 pairwise comparisons. Wrap the batch in a wall-clock timing harness (record start/stop times around the run) so per-run and total times can be written to a CSV. Run with default parameters as the baseline; the paper reports the 10 pairwise comparisons over the five serum datasets (42,629-47,626 spectra each) took 40 min on an Intel Xeon W-2135 at 3.70 GHz, so on different hardware expect a same-order-of-magnitude runtime rather than an exact match. As a sanity check, the article notes that 100 datasets (4,950 pairwise comparisons) take several hours, providing an anchor for scaling expectations.

## Related tools

- **compareMS2** (The main tool: computes distances between sets of tandem mass spectra; run in command-line mode for benchmarking and batch execution over the pairwise comparisons.) — https://github.com/524D/compareMS2.git
- **compareMS2 2.0** (The updated version under test, providing the default parameter set and the pairwise spectrum-set distance metric being timed.) — https://github.com/524D/compareMS2
- **Intel Xeon W-2135** (Reference CPU (3.70 GHz) on which the paper's 40 min baseline for 10 pairwise comparisons was measured.)

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn
```

## Evaluation signals

- Number of pairwise comparisons executed equals n(n-1)/2 for the input dataset count (10 for five datasets); each comparison appears once in the distance matrix.
- Wall-clock runtime is same-order-of-magnitude as the reported 40 min for five serum datasets on comparable hardware (Xeon W-2135 at 3.70 GHz), not an exact match across machines.
- The distance matrix is symmetric by construction (D(A,B) = D(B,A)), so reversed pairs need not be recomputed; each diagonal element D(A,A) is zero.
- Runtime CSV records start/stop timestamps and total time for the batch, allowing recalculation from raw timestamps.
- For larger batches, scaling is roughly O(n^2): the paper reports 100 datasets (4,950 pairwise comparisons) take several hours.

## Limitations

- Runtime is hardware-dependent: the 40 min baseline was measured on an Intel Xeon W-2135 (3.70 GHz); different CPUs will yield different wall-clock times, so only order-of-magnitude agreement is expected.
- The GUI draws a UPGMA tree only at completion of each row of the distance matrix; in CLI mode there is no real-time visual feedback, so you must wait for the batch to finish before inspecting results.
- The number of pairwise comparisons grows quadratically — comparing 100 LC-MS/MS datasets requires 4,950 pairwise comparisons and takes several hours.
- compareMS2 compares sets of tandem mass spectra rather than to predicted spectra of specific peptides, so it is not a substitute for database search or spectral library identification.

## Evidence

- [results] 10 pairwise comparisons took 40 min with default parameters on Intel Xeon W-2135: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [readme] Rationale for choosing CLI over GUI for batch execution: [evidence span withheld — anchor status: external_doc]
- [readme] D(A,B) symmetry justification — no need to recompute reversed pairs: [evidence span withheld — anchor status: external_doc]
- [results] Scaling expectation for larger batches: "comparing 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours"
- [discussion] Scope: spectrum-set comparison, not peptide identification: "compareMS2
compares sets of tandem mass spectra to each other rather
than to predicted spectra of specific peptides as when identifying
proteins from tandem mass spectra."
- [methods] Install via git clone and yarn: "git clone https://github.com/524D/compareMS2.git
cd compareMS2
corepack enable
yarn"
