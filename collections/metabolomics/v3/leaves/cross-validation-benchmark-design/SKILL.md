---
name: cross-validation-benchmark-design
description: Use when a claimed annotation-propagation algorithm must be evaluated
  for true generalization to metabolites lacking seeds — i.e., when you need a held-out
  benchmark of coverage and Top 1/3/10 correct rates rather than a single-shot annotation
  run.
when_to_use_negative:
- When you only need to annotate a fully unknown dataset with no set of validated
  known metabolite annotations to partition into seed/validation folds — the benchmark
  requires known ground-truth annotations.
- When the goal is absolute per-feature identification with chemical standards rather
  than relative comparison of annotation-propagation algorithms.
- When data come from non-Orbitrap instruments or lack MS2 spectra, since the 20-dataset
  benchmark and MS2-similarity-driven propagation assume Orbitrap-style MS2 data.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3
  role: Primary annotation engine being benchmarked; run per fold via MrnAnnoAlgo()
    with getMetAnno(), assignConfidenceInMrn3(), and rmRedunInMrn3() to propagate
    annotations across the two-layer network
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Baseline single-layer annotation tool against which MetDNA3's coverage and
    correct-rate improvements are measured
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:cfbf9f732e970545a48e4cfdce6303aff2069196eb14c38a3f80967c5e0b5818
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-validation-benchmark-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking protocol that partitions known metabolite annotations into 10 cross-validation folds (30% seeds / 70% validation targets) to quantify how much network-based annotation propagation (MetDNA3 vs MetDNA2) improves annotation coverage and correct rate on 20 Orbitrap untargeted metabolomics datasets.

## When to use

Use when a claimed annotation-propagation algorithm must be evaluated for true generalization to metabolites lacking seeds — i.e., when you need a held-out benchmark of coverage and Top 1/3/10 correct rates rather than a single-shot annotation run.

## When NOT to use

- When you only need to annotate a fully unknown dataset with no set of validated known metabolite annotations to partition into seed/validation folds — the benchmark requires known ground-truth annotations.
- When the goal is absolute per-feature identification with chemical standards rather than relative comparison of annotation-propagation algorithms.
- When data come from non-Orbitrap instruments or lack MS2 spectra, since the 20-dataset benchmark and MS2-similarity-driven propagation assume Orbitrap-style MS2 data.

## Inputs

- Supplementary Data 2: seed and validation metabolite annotations for the 20 Orbitrap datasets
- Source Data file for Fig. 3e–g (reported benchmark values)
- MS1 peak table (.csv with 'name', 'mz', 'rt' columns) and MS2 data (.mgf or .msp) per MetDNA2/MetDNA3 input spec
- Sample information table (.csv with 'sample.name' and 'group' columns)
- Metabolite database / metabolic reaction network used by MetDNA3

## Outputs

- cv_annotation_benchmark.csv with per-fold and aggregate annotation coverage, Top 1/3/10 correct rates, and Top 3 FDR
- Per-fold annotation result tables with confidence levels and redundancy removed
- Comparison of MetDNA3 vs MetDNA2 baseline coverage (68.1% vs 39.9%)

## How to apply

Load the seed/validation metabolite annotation table (Supplementary Data 2) and the Source Data for Fig. 3e–g, then split the known metabolite annotations into 10 cross-validation folds, holding 30% of metabolites as seeds and treating the remaining 70% as validation targets. For each fold, run MetDNA3 via MrnAnnoAlgo(): use getMetAnno() to recursively propagate annotations from the seed metabolites across the two-layer knowledge/data network, then assign confidence levels with assignConfidenceInMrn3() and remove redundant annotations with rmRedunInMrn3() on each fold's result table. Compute per-fold annotation coverage on the validation metabolites and compare against the MetDNA2 baseline (39.9% average coverage). Aggregate across all 10 folds to compute Top 1/3/10 correct rates and Top 3 FDR, writing per-fold plus aggregate metrics to cv_annotation_benchmark.csv. The rationale is that held-out validation metabolites receive annotations only through network propagation from the seed subset, so gains in coverage/accuracy reflect genuine propagation ability rather than direct library matching.

## Related tools

- **MetDNA3** (Primary annotation engine being benchmarked; run per fold via MrnAnnoAlgo() with getMetAnno(), assignConfidenceInMrn3(), and rmRedunInMrn3() to propagate annotations across the two-layer network) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Baseline single-layer annotation tool against which MetDNA3's coverage and correct-rate improvements are measured) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); anno <- getMetAnno(table_anno, idx_seeds, round, obj_mrn, table_ms2_network_filter, step_mrn, topN_neighbor, lib_type_mrn, thread)
```

## Evaluation signals

- Aggregated 10-fold coverage should reproduce the reported values: MetDNA3 68.1% vs MetDNA2 39.9%, within reasonable tolerance of the Source Data for Fig. 3e–g
- Aggregated Top 1/3/10 correct rates should be 68.0%/84.4%/91.0% and Top 3 FDR 15.6%
- Each fold must use exactly 30% of known metabolites as seeds and 70% as validation targets, with validation metabolites excluded from the seed set
- Per-fold result tables must have confidence levels assigned (assignConfidenceInMrn3) and redundant annotations removed (rmRedunInMrn3) before computing correct rates
- Per-fold metrics should be written to cv_annotation_benchmark.csv alongside the aggregate row for reproducibility checking

## Limitations

- Validation tools may have included part of the validation metabolites in their training sets, potentially inflating accuracy
- Cross-tool concordance may overestimate performance because chemical standards are lacking
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so FDR estimates from MS2 matching may be optimistic
- No FDR is reported for the curated MRN due to lack of clearly defined non-RPs and inapplicability of GNN performance metrics to the filtered candidate set
- MetDNA2's R package omits library objects (zhuMetLib, lib_rt, lib_ccs) for copyright reasons, so exact baseline reproduction requires user-supplied libraries

## Evidence

- [abstract] coverage benchmark vs MetDNA2 baseline: "Overall, this strategy significantly improves the coverage, accuracy, and efficiency of metabolite annotation and is freely available as MetDNA3."
- [intro] propagation method: "By leveraging cross-network interactions, recursive-based metabolite annotation propagation is efficiently achieved with 10-fold improved computational efficiency"
- [methods] confidence and redundancy functions: "- `rmRedunInMrn3(result_df, is_only_confidence_assignment)`"
