---
name: network-topology-comparison-analysis
description: Use when you must decide which knowledge-driven network topology (metabolic
  reaction network vs. spectral or fully connected network) to adopt for MS2-based
  annotation propagation, by quantifying accuracy, FDR, and specificity trade-offs.
when_to_use_negative:
- When the input is already a single-feature annotation result and no topology choice
  is being made — this skill benchmarks topologies, it does not annotate features
  directly.
- When the dataset is not LC-MS/MS Orbitrap untargeted metabolomics with MS2 spectra
  available, since decoy-MS2 evaluation and propagation specificity depend on MS2
  spectral data.
- When you need a single annotation result for downstream analysis rather than a comparative
  evaluation across network connectivities.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetDNA3
  role: Implements the recursive annotation propagation algorithm (MrnAnnoAlgo3 core)
    that is run under each topology (MRN/SMN/FCN) to generate candidate annotations
    for benchmarking
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: Builds and stores the three knowledge-driven network topologies (MRN, SMN,
    FCN) as graph objects for propagation and topology comparison
- name: readr
  role: Loads Supplementary Table 1 (dataset list) and Supplementary Data 4 (decoy
    MS2 entries) into R
- name: dplyr
  role: Aggregates the three metrics per topology into the summary table and writes
    benchmark_topology_metrics.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:8d0825955bd9d775480f3cc4685a849baac795cc781413dc3433404410586578
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
  - MetDNA3
  - dplyr
  - igraph
  - readr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# network-topology-comparison-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark different knowledge-driven network topologies (MRN, SMN, FCN) for network-based metabolite annotation by computing same-superclass propagation specificity, decoy-MS2 Top-3 accuracy, and estimated FDR per topology, revealing that sparser networks (MRN) outperform denser ones.

## When to use

Use when you must decide which knowledge-driven network topology (metabolic reaction network vs. spectral or fully connected network) to adopt for MS2-based annotation propagation, by quantifying accuracy, FDR, and specificity trade-offs.

## When NOT to use

- When the input is already a single-feature annotation result and no topology choice is being made — this skill benchmarks topologies, it does not annotate features directly.
- When the dataset is not LC-MS/MS Orbitrap untargeted metabolomics with MS2 spectra available, since decoy-MS2 evaluation and propagation specificity depend on MS2 spectral data.
- When you need a single annotation result for downstream analysis rather than a comparative evaluation across network connectivities.

## Inputs

- 20 Orbitrap untargeted metabolomics datasets (Supplementary Table 1 list)
- Decoy MS2 reference spectra entries (Supplementary Data 4)
- Three knowledge-driven network topologies — MRN, SMN, FCN — constructed from metabolite/metabolic-reaction data
- MetDNA3/MrnAnnoAlgo3 annotation outputs (candidate annotations per feature per topology)

## Outputs

- benchmark_topology_metrics.csv — per-topology summary table of same-superclass propagation specificity, decoy Top-3 accuracy, and estimated FDR
- Per-topology decoy-based accuracy and FDR estimates (reported reference values: specificity 80.8%/62.6%/24.9%, Top-3 accuracy 73.3%/65.1%/58.3%, FDR 15.8%/36.4%/44.5% for MRN/SMN/FCN)

## How to apply

For each dataset in the benchmark set (20 Orbitrap untargeted metabolomics datasets from Supplementary Table 1), run the MrnAnnoAlgo3 recursive annotation propagation under three knowledge-driven topologies — MRN (metabolic reaction network), SMN, and FCN — building each network as an igraph object. For each topology, compute the same-superclass propagation specificity (fraction of propagated annotation pairs sharing the same chemical superclass) and the decoy-based Top-3 accuracy, where candidate annotations are ranked against decoy MS2 reference spectra from Supplementary Data 4 and the true metabolite must rank within the top 3 candidates; from the decoy evaluation, estimate the annotation FDR per topology. Aggregate the three metrics per topology into a summary table with dplyr and export as benchmark_topology_metrics.csv. The rationale, grounded in the article, is that indiscriminately increasing network connectivity degrades annotation quality: reported benchmark values are same-superclass specificity 80.8%/62.6%/24.9%, decoy Top-3 accuracy 73.3%/65.1%/58.3%, and estimated FDR 15.8%/36.4%/44.5% for MRN/SMN/FCN, so the sparser MRN topology is preferred.

## Related tools

- **MetDNA3** (Implements the recursive annotation propagation algorithm (MrnAnnoAlgo3 core) that is run under each topology (MRN/SMN/FCN) to generate candidate annotations for benchmarking) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (Builds and stores the three knowledge-driven network topologies (MRN, SMN, FCN) as graph objects for propagation and topology comparison)
- **readr** (Loads Supplementary Table 1 (dataset list) and Supplementary Data 4 (decoy MS2 entries) into R)
- **dplyr** (Aggregates the three metrics per topology into the summary table and writes benchmark_topology_metrics.csv)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); library(MrnAnnoAlgo3); # run recursive annotation propagation under MRN, SMN, FCN topologies and compare metrics
```

## Evaluation signals

- Computed same-superclass propagation specificity per topology must match the reported values 80.8% (MRN), 62.6% (SMN), 24.9% (FCN) within reasonable numerical tolerance.
- Decoy-based Top-3 accuracy per topology must match the reported values 73.3% (MRN), 65.1% (SMN), 58.3% (FCN).
- Estimated FDR per topology must match the reported values 15.8% (MRN), 36.4% (SMN), 44.5% (FCN).
- The final summary table must contain exactly three rows (MRN, SMN, FCN) with all three metrics populated.
- Monotonic trend check: same-superclass specificity and Top-3 accuracy must decrease while estimated FDR must increase as connectivity increases from MRN → SMN → FCN.

## Limitations

- Benchmark is restricted to Orbitrap datasets; results may not transfer directly to other instrument platforms (e.g. SciexTripleTOF).
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives, which can affect the MRN topology construction.
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so FDR estimates from MS2 matching may be optimistic.
- The study refrains from reporting an FDR for the curated MRN itself, due to the lack of clearly defined non-RPs (negative examples) under the screening conditions.

## Evidence

- [results] increasing network connectivity from MRN to SMN to FCN degraded annotation accuracy (73.3% to 65.1% to 58.3%) and raised estimated FDR (15.8% to 36.4% to 44.5%): [evidence span withheld — anchor status: not_found]
- [results] the sparser MRN gave the highest same-superclass propagation specificity (80.8% vs 62.6% for SMN and 24.9% for FCN): [evidence span withheld — anchor status: not_found]
