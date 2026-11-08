---
name: ppm-mass-tolerance-configuration
description: Use when benchmarking formula ranking tools (SIRIUS, Buddy, FSA) on shared
  MS/MS datasets and each tool's mass tolerance must be set to its documented optimum
  (e.g., SIRIUS 5.8.6 with the paper's optimal ppm tolerance, Buddy 0.3.8 with 10-20
  ppm).
when_to_use_negative:
- When the goal is de novo method development rather than head-to-head tool benchmarking
  under published tolerance settings (e.g., FSA's own development, which tested additional
  scoring terms via a genetic algorithm rather than fixed tolerance configurations).
- When only one tool is being run without a comparative accuracy/runtime claim, since
  the tolerance-configuration skill exists to make cross-tool Table 3 comparisons
  reproducible.
edam_operation: http://edamontology.org/operation_3214
tools:
- name: SIRIUS
  role: Formula assignment tool run at version 5.8.6 with the paper's optimal ppm
    mass tolerance to produce ranked candidate molecular formulas per spectrum for
    accuracy/runtime comparison against FSA
- name: Buddy
  role: Formula assignment tool run at version 0.3.8 with a 10-20 ppm mass tolerance
    to produce ranked candidate molecular formulas per spectrum for the head-to-head
    benchmark
- name: FSA
  role: Baseline formula ranking method whose published rank-1 (85.28%), top-5 (97.35%),
    and 0.024 s/spectrum runtime values anchor the tolerance-configured comparison
  repo: kslynn128171/FSA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:23806b7a936cc172aa1aa88a225a352886a0ea868e6e790450cd9afaa3ceab6b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - Buddy
  - FSA
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ppm-mass-tolerance-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures per-tool mass tolerance (ppm) settings when benchmarking formula assignment tools such as SIRIUS, Buddy, and FSA on the same LC-MS/MS datasets, so that rank-1/top-3/top-5 accuracy and wall-clock runtime comparisons are run under each tool's documented optimal tolerance.

## When to use

Use when benchmarking formula ranking tools (SIRIUS, Buddy, FSA) on shared MS/MS datasets and each tool's mass tolerance must be set to its documented optimum (e.g., SIRIUS 5.8.6 with the paper's optimal ppm tolerance, Buddy 0.3.8 with 10-20 ppm).

## When NOT to use

- When the goal is de novo method development rather than head-to-head tool benchmarking under published tolerance settings (e.g., FSA's own development, which tested additional scoring terms via a genetic algorithm rather than fixed tolerance configurations).
- When only one tool is being run without a comparative accuracy/runtime claim, since the tolerance-configuration skill exists to make cross-tool Table 3 comparisons reproducible.

## Inputs

- Eight qualified MS/MS test datasets with ground-truth reference molecular formulas per spectrum
- Tool installation versions: SIRIUS 5.8.6, Buddy 0.3.8, FSA
- Documented mass tolerance settings: the paper's optimal ppm tolerance for SIRIUS, 10-20 ppm for Buddy

## Outputs

- Ranked candidate molecular formula lists per spectrum per tool
- Per-dataset rank-1, top-3, and top-5 accuracy values per tool
- Total and per-spectrum wall-clock processing times recorded on identical hardware
- Comparison table reproducing published Table 3 values and FSA-vs-tool speed ratios (~148x Buddy, ~134x SIRIUS)

## How to apply

For each candidate formula-ranking tool, set the mass tolerance to the value recommended by its own evaluation rather than reusing one tolerance across tools: SIRIUS 5.8.6 is run with the paper's optimal ppm mass tolerance, while Buddy 0.3.8 is run with a 10-20 ppm mass tolerance. Apply the configured tolerance identically across all eight qualified MS/MS test datasets, producing ranked candidate molecular formulas per spectrum. Then compute rank-1, top-3, and top-5 accuracy per dataset by checking whether the ground-truth reference formula falls within the corresponding rank window, and record total and per-spectrum wall-clock time on the same hardware. The rationale is that accuracy rankings (e.g., FSA's 74.01% vs Buddy's 97.63% rank#1) and speed ratios (~148x over Buddy, ~134x over SIRIUS, 12.5 min vs 1848/1678 min) are only reproducible when each tool runs under its published tolerance configuration.

## Related tools

- **SIRIUS** (Formula assignment tool run at version 5.8.6 with the paper's optimal ppm mass tolerance to produce ranked candidate molecular formulas per spectrum for accuracy/runtime comparison against FSA)
- **Buddy** (Formula assignment tool run at version 0.3.8 with a 10-20 ppm mass tolerance to produce ranked candidate molecular formulas per spectrum for the head-to-head benchmark)
- **FSA** (Baseline formula ranking method whose published rank-1 (85.28%), top-5 (97.35%), and 0.024 s/spectrum runtime values anchor the tolerance-configured comparison) — kslynn128171/FSA

## Evaluation signals

- Per-dataset rank-1/top-3/top-5 accuracies and runtime entries must match the published Table 3 values for SIRIUS and Buddy within the paper's reporting granularity
- Computed FSA-vs-tool speed ratios must reproduce ~148x over Buddy and ~134x over SIRIUS (12.5 min vs 1848/1678 min)
- FSA baseline must show ~85.28% rank#1 and ~97.35% top-5 accuracy across 30,690 MS/MS spectra from 6792 metabolites
- FSA average processing time per spectrum must be ~0.024 s, consistent with the published runtime
- Each tool's tolerance setting must be documented and held constant across all eight datasets before accuracy computation

## Limitations

- FSA's rank#1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS because it does not utilize structural information; tolerance configuration does not close this accuracy gap, only makes comparison fair.
- Few chemical rules are implemented to rank or filter out unfeasible formulas, and species/adduct-specific rules are not yet incorporated, which can bias formula ranking regardless of tolerance setting.
- FSA relies on a PubChem snapshot of 109,383,767 compounds vs the current 112,419,402 compounds as of 2022/11/27, so candidate formula inputs differ from a current-database run.

## Evidence
