---
name: metabolomics-annotation-pipeline-comparison
description: Use when comparing computational efficiency of annotation pipelines (e.g.,
  MetDNA3 vs MetDNA2) on the same metabolomics datasets, or when validating a claimed
  fold speedup from published runtime tables.
when_to_use_negative:
- When comparing annotation coverage, accuracy, or FDR of tools rather than computational
  efficiency — runtime benchmarks do not inform annotation quality claims
- When the goal is a head-to-head accuracy comparison involving tools like GNPS/FBMN,
  IIMN, or NetID on benchmark sets with chemical standards
- When only a single dataset is available and no per-dataset runtime table exists
  — the mean and fold speedup require the 12-dataset/3-sample structure
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MetDNA3
  role: Faster annotation pipeline under benchmark; its recursive annotation propagation
    is timed per dataset and spot-checked via MrnAnnoAlgo3
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Baseline pipeline whose wall-clock runtime (~1082 min/dataset mean) establishes
    the comparison reference
  repo: https://github.com/ZhuMetLab/MetDNA2
- name: devtools
  role: Used to install MrnAnnoAlgo3 from GitHub for the runtime spot-check
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:bdf0cf64368c207d6f013f768e68ec01c654dcbf22aea8973fe991c2bb056bf4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA2
  - MetDNA3
  - devtools
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolomics-annotation-pipeline-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-clock runtime of MetDNA3's recursive annotation propagation against MetDNA2 across biological metabolomics datasets, quantifying the reported >14-fold per-dataset speedup (mean 77 min vs 1082 min) using the Source Data for Fig. 2g.

## When to use

Use when comparing computational efficiency of annotation pipelines (e.g., MetDNA3 vs MetDNA2) on the same metabolomics datasets, or when validating a claimed fold speedup from published runtime tables.

## When NOT to use

- When comparing annotation coverage, accuracy, or FDR of tools rather than computational efficiency — runtime benchmarks do not inform annotation quality claims
- When the goal is a head-to-head accuracy comparison involving tools like GNPS/FBMN, IIMN, or NetID on benchmark sets with chemical standards
- When only a single dataset is available and no per-dataset runtime table exists — the mean and fold speedup require the 12-dataset/3-sample structure

## Inputs

- Source Data file for Fig. 2g (runtime table, DOI 10.1038/s41467-025-63536-6)
- MetDNA3 demo dataset from the MetDNA webserver (MS1 peak table .csv, MS2 .mgf/.msp, sample info table)
- MetDNA2 demo projects (e.g., NIST urine OEP003157, fruit fly MTBLS612) with documented runtimes

## Outputs

- Per-tool mean per-dataset runtime summary (MetDNA2 ~1082 min, MetDNA3 ~77 min)
- Fold speedup statistic (>14x)
- Grouped bar chart of per-dataset runtimes replicating Fig. 2g (summary and figure files written)

## How to apply

Download the Source Data file for Fig. 2g (DOI 10.1038/s41467-025-63536-6) and load the runtime table with pandas; the table should contain wall-clock runtimes for MetDNA2 and MetDNA3 over 12 datasets drawn from 3 biological samples. Drop incomplete entries, then aggregate per-tool runtimes and compute the mean per-dataset runtime for each tool — expect approximately 1082 min for MetDNA2 and 77 min for MetDNA3 — and the fold speedup, which should exceed 14x. Spot-check the MetDNA3 claim empirically by installing the MrnAnnoAlgo3 R package from GitHub with devtools and re-running the recursive annotation propagation on one demo dataset from the MetDNA webserver (metdna.zhulab.cn), confirming a typical dataset completes in about one hour; the MetDNA2 README similarly documents per-project runtimes of roughly 4-8 hours (e.g., NIST urine Pos 5.4 h, Neg 8.8 h) that can anchor the slow baseline. Finally, plot the per-dataset runtimes as a grouped bar chart with matplotlib to replicate Fig. 2g.

## Related tools

- **MetDNA3** (Faster annotation pipeline under benchmark; its recursive annotation propagation is timed per dataset and spot-checked via MrnAnnoAlgo3) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Baseline pipeline whose wall-clock runtime (~1082 min/dataset mean) establishes the comparison reference) — https://github.com/ZhuMetLab/MetDNA2
- **devtools** (Used to install MrnAnnoAlgo3 from GitHub for the runtime spot-check)

## Examples

```
import pandas as pd; df = pd.read_csv('Fig2g_runtime.csv').dropna(); mean = df.groupby('tool')['runtime_min'].mean(); print(1082 / 77)
```

## Evaluation signals

- Computed mean per-dataset runtime for MetDNA2 matches ~1082 min and MetDNA3 ~77 min within rounding tolerance
- Computed fold speedup exceeds 14x, consistent with the reported >14-fold improvement
- Runtime table contains 12 datasets from 3 biological samples with complete entries for both tools after dropping incomplete rows
- Independent re-run of MrnAnnoAlgo3 on one webserver demo dataset completes in approximately one hour
- Summary file and grouped bar chart figure are written and visually match Fig. 2g

## Limitations

- Runtime benchmarks are hardware- and configuration-dependent; absolute times may not transfer across machines
- The paper claims 'over 10-fold improved computational efficiency' in the abstract while the Fig. 2g analysis yields >14-fold — quoting the speedup should name the specific comparison
- MetDNA2 README notes runtimes depend on the number of features and MS/MS spectra (4-8 hours per project), so dataset size confounds direct per-dataset comparison
- The MrnAnnoAlgo3 README cautions that 'Full MetDNA3 functionality requires additional modules', so the R package alone reproduces only the core algorithm timing, not the full webserver workflow

## Evidence

- [readme] Generally, it requires 4-8 hours to complete a project, which depends on the number of features and MS/MS spectra.: [evidence span withheld — anchor status: external_doc]
