---
name: computational-runtime-benchmarking
description: Use when quantifying the computational-efficiency gain of an updated
  annotation pipeline over its predecessor, e.g. MetDNA3 vs MetDNA2 runtimes (77 min
  vs 1082 min per dataset) over the same 12 datasets.
when_to_use_negative:
- Input is annotation accuracy/coverage data rather than wall-clock runtimes — use
  the annotation-performance comparison workflow instead.
- Only one pipeline version's runtime is available with no matched baseline (MetDNA2
  runtimes) for the same datasets, so a fold-speedup cannot be computed.
- Dataset runtimes come from different hardware or different feature counts, making
  the wall-clock comparison invalid.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: MetDNA3
  role: Newer two-layer networking annotation pipeline whose runtime is the measured
    quantity
  repo: github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Baseline knowledge-guided multi-layer pipeline providing reference runtimes
    (5.0-8.8 h per project per its README)
  repo: https://github.com/ZhuMetLab/MetDNA2
- name: MrnAnnoAlgo3
  role: Core MetDNA3 algorithm module installed via devtools and run on a demo dataset
    for the runtime spot-check
  repo: github.com/ZhuMetLab/MrnAnnoAlgo3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:967e9f38c1bc663087c8f855cc4df7e3c0f28bf08e0326bf6e9c3e46364064c5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA2
  - MetDNA3
  - MrnAnnoAlgo3
  - devtools
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# computational-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks wall-clock runtime of two metabolite-annotation pipelines (MetDNA2 vs MetDNA3) across a fixed panel of biological datasets and reports the per-dataset mean runtime and fold speedup, replicating the paper's Fig. 2g efficiency comparison.

## When to use

Use when quantifying the computational-efficiency gain of an updated annotation pipeline over its predecessor, e.g. MetDNA3 vs MetDNA2 runtimes (77 min vs 1082 min per dataset) over the same 12 datasets.

## When NOT to use

- Input is annotation accuracy/coverage data rather than wall-clock runtimes — use the annotation-performance comparison workflow instead.
- Only one pipeline version's runtime is available with no matched baseline (MetDNA2 runtimes) for the same datasets, so a fold-speedup cannot be computed.
- Dataset runtimes come from different hardware or different feature counts, making the wall-clock comparison invalid.

## Inputs

- Source Data runtime table for Fig. 2g (wall-clock runtimes for MetDNA2 and MetDNA3, 12 datasets from 3 biological samples)
- MetDNA3 demo dataset from the MetDNA webserver (http://metdna.zhulab.cn) for the MrnAnnoAlgo3 spot-check

## Outputs

- Mean per-dataset runtime summary (MetDNA2 ~1082 min vs MetDNA3 ~77 min) with >14x fold speedup
- Grouped bar chart of per-dataset runtimes replicating Fig. 2g
- Spot-check runtime report for one MrnAnnoAlgo3 demo-dataset run (~1 hour)

## How to apply

Download the Source Data file backing the runtime panel figure (Fig. 2g, DOI 10.1038/s41467-025-63536-6) and load the wall-clock runtime table with pandas. Verify the table covers 12 datasets drawn from 3 biological samples with per-dataset runtimes for both MetDNA2 and MetDNA3, dropping incomplete entries. Aggregate per-tool runtimes, compute the mean per-dataset runtime for each tool (expected ~1082 min for MetDNA2 vs ~77 min for MetDNA3) and the fold speedup (expected >14x). Optionally spot-check the MetDNA3 claim by installing MrnAnnoAlgo3 from GitHub with devtools and re-running the MrnAnnoAlgo3 pipeline on one demo dataset from the MetDNA webserver, confirming a typical dataset completes in about one hour. Finally, plot per-dataset runtimes as a grouped bar chart with matplotlib to replicate Fig. 2g, and write summary and figure files. Rationale: the paper's two-layer networking topology (knowledge-driven MRN plus data-driven MS2 similarity network) enables interactive recursive annotation propagation, which is what drives the efficiency gain over MetDNA2's multi-layer propagation.

## Related tools

- **MetDNA3** (Newer two-layer networking annotation pipeline whose runtime is the measured quantity) — github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Baseline knowledge-guided multi-layer pipeline providing reference runtimes (5.0-8.8 h per project per its README)) — https://github.com/ZhuMetLab/MetDNA2
- **MrnAnnoAlgo3** (Core MetDNA3 algorithm module installed via devtools and run on a demo dataset for the runtime spot-check) — github.com/ZhuMetLab/MrnAnnoAlgo3

## Examples

```
if (!require("devtools")) install.packages("devtools"); devtools::install_github("ZhuMetLab/MrnAnnoAlgo3")
```

## Evaluation signals

- Computed mean runtime for MetDNA2 is approximately 1082 minutes across the 12 datasets
- Computed mean runtime for MetDNA3 is approximately 77 minutes, matching the reported value within rounding tolerance
- Fold speedup exceeds 14x, consistent with the article's stated >10-fold efficiency improvement
- Runtime table rows are complete for all 12 datasets from 3 biological samples after dropping incomplete entries
- MrnAnnoAlgo3 spot-check on one demo dataset completes in roughly one hour, matching the README's 'one hour' claim; summary and figure files are written

## Limitations

- The paper refrains from reporting an FDR for the curated MRN, and this skill benchmarks runtime only — accuracy and coverage are not validated by a runtime comparison.
- MetDNA2's R package removes the library objects (zhuMetLib, zhuMetlib_orbitrap, zhuRPlib, lib_rt, lib_ccs) 'Due to the limitation of copyright', so a local re-run of the MetDNA2 baseline may not be directly reproducible without those libraries.
- Full MetDNA3 functionality requires additional modules beyond MrnAnnoAlgo3 (available via the MetDNA3 webserver with free registration), so the spot-check measures only the core algorithm module.

## Evidence

- [abstract] The generated networking topology enables interactive annotation propagation with over 10-fold improved computational efficiency.: "The generated networking topology enables interactive annotation propagation with over 10-fold improved computational efficiency."
- [intro] By leveraging cross-network interactions, recursive-based metabolite annotation propagation is efficiently achieved with 10-fold improved computational efficiency: "By leveraging cross-network interactions, recursive-based metabolite annotation propagation is efficiently achieved with 10-fold improved computational efficiency"
- [readme] Processes a typical untargeted metabolomics dataset in just one hour—over 10-fold faster than previous versions.: "Processes a typical untargeted metabolomics dataset in just one hour—over **10-fold faster** than previous versions."
- [readme] Generally, it requires 4-8 hours to complete a project, which depends on the number of features and MS/MS spectra.: [evidence span withheld — anchor status: external_doc]
