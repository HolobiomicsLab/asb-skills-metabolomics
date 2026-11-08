---
name: spectral-tool-runtime-benchmarking
description: Use when comparing formula assignment tools (e.g., FSA vs SIRIUS vs Buddy)
  on identical MS/MS datasets and you need paired rank-1/top-3/top-5 accuracy plus
  total and per-spectrum runtime to quantify accuracy-speed trade-offs.
when_to_use_negative:
- The datasets lack ground-truth reference formulas, so rank-1/top-3/top-5 accuracy
  cannot be computed against known correct formulas
- The goal is metabolite identification with structural information rather than formula-level
  ranking accuracy and runtime benchmarking
- You need a single-tool quality assessment rather than a controlled same-hardware,
  same-dataset head-to-head comparison
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FSA
  role: 'Skill under benchmarking: its published accuracy (74.01% rank#1) and runtime
    (12.5 min across the eight datasets; 0.024 s per spectrum) serve as the reference
    baseline against which SIRIUS and Buddy results are compared'
  repo: kslynn128171/FSA
- name: SIRIUS
  role: Baseline competitor run as SIRIUS 5.8.6 with the paper's optimal ppm mass
    tolerance to produce ranked candidate molecular formulas per spectrum
- name: Buddy
  role: Baseline competitor run as Buddy 0.3.8 with 10-20 ppm mass tolerance to produce
    ranked candidate molecular formulas per spectrum
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:d20205258abb2eee624a5023c81cc7fdc1f96bb1d2697400af0f1c2f8e13ea29
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
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# spectral-tool-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A paired head-to-head benchmarking protocol that measures molecular formula ranking accuracy (rank#1, top-3, top-5) and wall-clock processing time for formula assignment tools (FSA, SIRIUS, Buddy) on the same MS/MS test datasets, quantifying the accuracy-versus-speed trade-off.

## When to use

Use when comparing formula assignment tools (e.g., FSA vs SIRIUS vs Buddy) on identical MS/MS datasets and you need paired rank-1/top-3/top-5 accuracy plus total and per-spectrum runtime to quantify accuracy-speed trade-offs.

## When NOT to use

- The datasets lack ground-truth reference formulas, so rank-1/top-3/top-5 accuracy cannot be computed against known correct formulas
- The goal is metabolite identification with structural information rather than formula-level ranking accuracy and runtime benchmarking
- You need a single-tool quality assessment rather than a controlled same-hardware, same-dataset head-to-head comparison

## Inputs

- Eight qualified MS/MS test datasets (LC-MS/MS spectra from diverse sample types)
- Per-dataset ground-truth reference molecular formulas
- SIRIUS 5.8.6 run with the paper's optimal ppm mass tolerance
- Buddy 0.3.8 run with 10-20 ppm mass tolerance
- FSA's published accuracy and runtime values (e.g., Table 3) for the comparison table

## Outputs

- Per-dataset rank-1, top-3, and top-5 accuracy values for each tool (SIRIUS, Buddy, FSA)
- Total and per-spectrum wall-clock processing times per tool on identical hardware
- Head-to-head comparison table with computed speed ratios (e.g., ~148x faster FSA vs Buddy, ~134x vs SIRIUS)

## How to apply

Load the eight qualified MS/MS test datasets together with each dataset's ground-truth reference formulas. Run SIRIUS 5.8.6 with the paper's optimal ppm mass tolerance and Buddy 0.3.8 with the 10-20 ppm mass tolerance on every dataset so each produces ranked candidate molecular formulas per spectrum. For each tool, compute rank-1, top-3, and top-5 accuracy per dataset by checking whether the correct formula falls within the corresponding rank window. Record each tool's total and per-spectrum wall-clock processing time on the same hardware, then assemble per-dataset accuracy and runtime entries into a comparison table alongside FSA's published values (e.g., FSA 74.01% rank#1, 12.5 min total vs Buddy 1848 min and SIRIUS 1678 min). Validate by verifying reproduction of the reported speed ratios (~148x over Buddy, ~134x over SIRIUS) and the Table 3 accuracy values at the paper's reporting granularity.

## Related tools

- **FSA** (Skill under benchmarking: its published accuracy (74.01% rank#1) and runtime (12.5 min across the eight datasets; 0.024 s per spectrum) serve as the reference baseline against which SIRIUS and Buddy results are compared) — kslynn128171/FSA
- **SIRIUS** (Baseline competitor run as SIRIUS 5.8.6 with the paper's optimal ppm mass tolerance to produce ranked candidate molecular formulas per spectrum)
- **Buddy** (Baseline competitor run as Buddy 0.3.8 with 10-20 ppm mass tolerance to produce ranked candidate molecular formulas per spectrum)

## Evaluation signals

- Per-dataset rank-1/top-3/top-5 accuracies for SIRIUS and Buddy match the published Table 3 values within the paper's reporting granularity
- Aggregate rank#1 accuracies reproduce the reported values: Buddy 97.63%, SIRIUS 70.31%, FSA 74.01%
- Computed speed ratios reproduce ~148x (FSA vs Buddy) and ~134x (FSA vs SIRIUS), consistent with 12.5 min FSA vs 1848 min (Buddy) and 1678 min (SIRIUS) totals
- Tool versions are pinned and identical (SIRIUS 5.8.6, Buddy 0.3.8) and all tools are timed on the same hardware
- FSA's per-spectrum processing time (0.024 s) is consistent with the total runtime recorded for the eight datasets

## Limitations

- FSA does not use structural information, so its rank#1 accuracy is not expected to be competitive with state-of-the-art tools such as Buddy and SIRIUS; the benchmark highlights speed, not accuracy superiority
- Results are tied to specific tool versions (SIRIUS 5.8.6, Buddy 0.3.8), fixed ppm tolerances (10-20 ppm for Buddy), and a single hardware configuration; re-benchmarking on other setups may shift ratios
- The comparison covers only the eight qualified test datasets and compounds composed of C, H, O, N, S, and P, so generalization to other element spaces or adduct types is untested

## Evidence

- [full_text] Buddy achieved the highest accuracy (97.63% rank#1), SIRIUS 70.31%, and FSA 74.01% rank#1 but FSA was ~148x faster than Buddy and ~134x faster than SIRIUS: [evidence span withheld — anchor status: not_found]
- [full_text] Runtime recording protocol: [evidence span withheld — anchor status: not_found]
- [full_text] Buddy mass tolerance specification: [evidence span withheld — anchor status: not_found]
