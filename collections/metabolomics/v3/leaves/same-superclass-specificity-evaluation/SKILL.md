---
name: same-superclass-specificity-evaluation
description: Use when benchmarking network-propagation annotation (e.g. MetDNA3/MrnAnnoAlgo3)
  across alternative knowledge-driven topologies, to test whether higher connectivity
  improves or degrades propagation specificity and decoy-based accuracy/FDR.
when_to_use_negative:
- When the goal is annotating a single dataset rather than comparing annotation strategies
  — the skill is a benchmarking/evaluation method, not an annotation routine
- When no chemical superclass taxonomy is available for the metabolites, since the
  specificity metric cannot be computed
- When no decoy MS2 spectra exist, since the paired FDR/Top-3 accuracy evaluation
  (15.8%/36.4%/44.5% FDR for MRN/SMN/FCN) cannot be replicated
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: MetDNA3
  role: Runs the recursive annotation propagation under each knowledge-driven topology
    (via the MrnAnnoAlgo3 core module) whose outputs are scored for same-superclass
    specificity
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: Builds and manipulates the MRN, SMN, and FCN network topologies so edge density/connectivity
    can be varied and compared
- name: MetDNA2
  role: Prior-generation R package whose igraph/dplyr/tidyr dependency stack and multi-layer
    metabolic network workflow provide the computational context for the propagation
    runs
  repo: https://github.com/ZhuMetLab/MetDNA2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:d431c158d6371062f52b228ea9758d7dca0953d99cd3ec618db79729808a5175
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
  - dplyr
  - igraph
  - readr
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Same-superclass specificity evaluation of network-based metabolite annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies annotation-propagation specificity by measuring the fraction of propagated annotation pairs that share the same chemical superclass across knowledge-driven network topologies (MRN, SMN, FCN) built for MetDNA3. It benchmarks whether increasing network connectivity helps or harms annotation quality.

## When to use

Use when benchmarking network-propagation annotation (e.g. MetDNA3/MrnAnnoAlgo3) across alternative knowledge-driven topologies, to test whether higher connectivity improves or degrades propagation specificity and decoy-based accuracy/FDR.

## When NOT to use

- When the goal is annotating a single dataset rather than comparing annotation strategies — the skill is a benchmarking/evaluation method, not an annotation routine
- When no chemical superclass taxonomy is available for the metabolites, since the specificity metric cannot be computed
- When no decoy MS2 spectra exist, since the paired FDR/Top-3 accuracy evaluation (15.8%/36.4%/44.5% FDR for MRN/SMN/FCN) cannot be replicated

## Inputs

- Untargeted metabolomics feature tables from the 20 Orbitrap datasets (Supplementary Table 1), with MS1 peak list (name, mz, rt columns) and MS2 spectra (.mgf/.msp)
- Knowledge-driven network definitions for the three topologies (MRN, SMN, FCN) to be built with igraph
- Chemical superclass annotations for metabolites in the candidate set
- Decoy MS2 reference entries (Supplementary Data 4) for decoy-based accuracy and FDR estimation

## Outputs

- Per-topology same-superclass propagation specificity values (fraction of propagated annotation pairs sharing a chemical superclass)
- Decoy-based Top 3 accuracy and decoy-derived estimated FDR per topology
- Aggregated summary table (benchmark_topology_metrics.csv) with metrics for MRN, SMN, and FCN

## How to apply

Build the three knowledge-driven topologies — metabolic reaction network (MRN), a denser SMN variant, and the fully connected FCN — as igraph graphs, then run MetDNA3's recursive annotation propagation (MrnAnnoAlgo3) separately under each topology on 20 Orbitrap untargeted metabolomics datasets. For each topology, compute the same-superclass propagation specificity as the fraction of propagated annotation pairs whose two metabolites share the same chemical superclass, and in parallel compute decoy-based Top 3 accuracy by ranking candidate annotations against decoy MS2 spectra and checking whether the true metabolite falls within the top 3, then derive decoy-based FDR estimates. Aggregate the three metrics per topology with dplyr into a summary table (benchmark_topology_metrics.csv). The rationale is that a sparser network should propagate annotations more conservatively: the reported benchmark shows specificity of 80.8% (MRN) vs 62.6% (SMN) vs 24.9% (FCN), decoy Top 3 accuracy of 73.3% vs 65.1% vs 58.3%, and estimated FDR of 15.8% vs 36.4% vs 44.5%.

## Related tools

- **MetDNA3** (Runs the recursive annotation propagation under each knowledge-driven topology (via the MrnAnnoAlgo3 core module) whose outputs are scored for same-superclass specificity) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (Builds and manipulates the MRN, SMN, and FCN network topologies so edge density/connectivity can be varied and compared)
- **MetDNA2** (Prior-generation R package whose igraph/dplyr/tidyr dependency stack and multi-layer metabolic network workflow provide the computational context for the propagation runs) — https://github.com/ZhuMetLab/MetDNA2

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); library(igraph)
```

## Evaluation signals

- Computed same-superclass specificity must reproduce the reported values (80.8% for MRN, 62.6% for SMN, 24.9% for FCN) within reasonable numerical tolerance
- Decoy-based Top 3 accuracy must match 73.3%/65.1%/58.3% and estimated FDR must match 15.8%/36.4%/44.5% for MRN/SMN/FCN
- Specificity must decrease monotonically as topology connectivity increases (MRN > SMN > FCN), confirming that indiscriminate connectivity does not improve annotation quality
- benchmark_topology_metrics.csv must contain one row/metric per topology with all three metrics (specificity, Top-3 accuracy, FDR) populated
- Per-dataset metrics (20 Orbitrap datasets) should be aggregated consistently, with no datasets dropped silently

## Limitations

- Fully connected topologies severely degrade propagation specificity (24.9% for FCN), so results cannot be extrapolated as 'more connectivity is better'
- No FDR is reported for the curated MRN itself, due to the lack of clearly defined non-RPs (negative examples) and inapplicability of GNN performance metrics to the filtered candidate set
- MS2 spectra often lack sufficient fragment information and isomeric metabolites may be indistinguishable, so decoy-based FDR estimates and superclass-based validation may be optimistic
- The MrnAnnoAlgo3 package is licensed CC BY-NC-ND 4.0 (non-commercial, no derivatives), and full MetDNA3 functionality requires webserver registration at http://metdna.zhulab.cn/

## Evidence

- [results] increasing network connectivity from MRN to SMN to FCN degraded annotation accuracy (73.3% to 65.1% to 58.3%) and raised estimated FDR (15.8% to 36.4% to 44.5%): [evidence span withheld — anchor status: not_found]
