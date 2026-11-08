---
name: metabolite-annotation-propagation-benchmarking
description: Use when you must quantify how the connectivity/edge density of a knowledge-driven
  metabolite network (MRN vs SMN vs FCN) changes annotation accuracy, estimated FDR,
  and propagation specificity, with decoy MS2 spectra as negative controls.
when_to_use_negative:
- Your goal is to increase annotation coverage by densifying the network — the benchmark
  shows added connectivity (SMN, FCN) raises estimated FDR (36.4%, 44.5%) and lowers
  specificity, so it is not a quality improvement
- Your MS2 spectra are not matched against decoy references and you lack the supplementary
  decoy entries (Supplementary Data 4), so FDR cannot be estimated
- You are annotating with a single fixed topology and need no comparative topology
  evaluation — use the standard MetDNA3 annotation workflow instead of this benchmarking
  skill
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA3 / MrnAnnoAlgo3
  role: Core algorithm module whose recursive annotation propagation is run under
    MRN, SMN, and FCN topologies for benchmarking
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Predecessor knowledge-guided multi-layer network package whose library objects
    and runMetDNA2 workflow contextualize the annotation propagation being benchmarked
  repo: https://github.com/ZhuMetLab/MetDNA2
- name: igraph
  role: Constructs and analyzes the MRN/SMN/FCN network topologies being compared
- name: readr
  role: Loads Supplementary Table 1 dataset list and Supplementary Data 4 decoy MS2
    entries
- name: dplyr
  role: Aggregates accuracy, FDR, and specificity metrics per topology into the summary
    CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:1a2133344a020afc49c6adac685923ecaa11904943c5cb515b0d246bd532e236
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
  - MetDNA3 / MrnAnnoAlgo3
  - dplyr
  - igraph
  - readr
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# metabolite-annotation-propagation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark how knowledge-driven network topologies of varying connectivity (MRN, SMN, FCN) affect network-based metabolite annotation quality in MetDNA3/MrnAnnoAlgo3, using decoy-MS2 evaluation on 20 Orbitrap untargeted metabolomics datasets to measure Top-3 accuracy, estimated FDR, and same-superclass propagation specificity.

## When to use

Use when you must quantify how the connectivity/edge density of a knowledge-driven metabolite network (MRN vs SMN vs FCN) changes annotation accuracy, estimated FDR, and propagation specificity, with decoy MS2 spectra as negative controls.

## When NOT to use

- Your goal is to increase annotation coverage by densifying the network — the benchmark shows added connectivity (SMN, FCN) raises estimated FDR (36.4%, 44.5%) and lowers specificity, so it is not a quality improvement
- Your MS2 spectra are not matched against decoy references and you lack the supplementary decoy entries (Supplementary Data 4), so FDR cannot be estimated
- You are annotating with a single fixed topology and need no comparative topology evaluation — use the standard MetDNA3 annotation workflow instead of this benchmarking skill

## Inputs

- Untargeted metabolomics feature tables with MS2 spectra from 20 Orbitrap datasets (Supplementary Table 1)
- Decoy MS2 reference entries (Supplementary Data 4)
- Knowledge-driven network topologies: MRN (metabolic reaction network), SMN, FCN (built with igraph)

## Outputs

- benchmark_topology_metrics.csv — per-topology summary of decoy Top-3 accuracy, estimated FDR, and same-superclass propagation specificity
- Per-topology annotation tables from the MrnAnnoAlgo3 recursive propagation run

## How to apply

Load the 20 Orbitrap untargeted metabolomics datasets (Supplementary Table 1) plus decoy MS2 reference entries (Supplementary Data 4) into R with readr. For each dataset, run the MrnAnnoAlgo3/MetDNA3 recursive annotation propagation (getMetAnno/getAdductAnno pipeline) separately under the three knowledge-driven topologies, building each network with igraph. Compute same-superclass propagation specificity as the fraction of propagated annotation pairs sharing the same chemical superclass, compute decoy-based Top-3 accuracy by checking whether the true metabolite ranks in the top 3 candidates when ranked against decoy MS2 spectra, and derive estimated FDR from the decoy results. Aggregate the three metrics per topology (MRN, SMN, FCN) with dplyr and validate against the reported benchmark values; the key decision point is that the sparser MRN should outperform, since indiscriminately increasing connectivity degrades annotation quality.

## Related tools

- **MetDNA3 / MrnAnnoAlgo3** (Core algorithm module whose recursive annotation propagation is run under MRN, SMN, and FCN topologies for benchmarking) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Predecessor knowledge-guided multi-layer network package whose library objects and runMetDNA2 workflow contextualize the annotation propagation being benchmarked) — https://github.com/ZhuMetLab/MetDNA2
- **igraph** (Constructs and analyzes the MRN/SMN/FCN network topologies being compared)
- **readr** (Loads Supplementary Table 1 dataset list and Supplementary Data 4 decoy MS2 entries)
- **dplyr** (Aggregates accuracy, FDR, and specificity metrics per topology into the summary CSV)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); library(MrnAnnoAlgo3); res <- getMetAnno(table_anno, idx_seeds, round, obj_mrn, table_ms2_network_filter, step_mrn, topN_neighbor, lib_type_mrn, thread)
```

## Evaluation signals

- Computed decoy Top-3 accuracy per topology matches reported values (MRN 73.3%, SMN 65.1%, FCN 58.3%) within numerical tolerance
- Estimated FDR matches reported values (MRN 15.8%, SMN 36.4%, FCN 44.5%)
- Same-superclass propagation specificity matches reported values (MRN 80.8%, SMN 62.6%, FCN 24.9%)
- Monotonic degradation of accuracy and specificity as connectivity increases MRN → SMN → FCN (a physical sanity invariant of the benchmark)
- Output benchmark_topology_metrics.csv contains exactly three rows (MRN/SMN/FCN) with all three metrics populated for each of the 20 datasets aggregated

## Limitations

- Isomeric metabolites may be indistinguishable based on MS2 spectra alone, so network-based annotation propagation and decoy-MS2 FDR estimates may be optimistic
- No FDR is reported for the curated MRN due to the lack of clearly defined non-RPs (negative examples), and GNN performance metrics are inapplicable to the filtered candidate set
- GNN-based direct prediction of reaction pairs may include a considerable number of false positives, and cross-tool concordance may overestimate performance due to the lack of chemical standards
- Full MetDNA3 functionality requires the webserver (metdna.zhulab.cn); the MrnAnnoAlgo3 GitHub package is a core module and MetDNA2's spectral library objects are removed from the R package for copyright reasons

## Evidence

- [results] increasing network connectivity from MRN to SMN to FCN degraded annotation accuracy (73.3% to 65.1% to 58.3%) and raised estimated FDR (15.8% to 36.4% to 44.5%): [evidence span withheld — anchor status: not_found]
