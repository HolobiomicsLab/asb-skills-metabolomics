---
name: untargeted-metabolomics-dataset-processing
description: Use when a batch of untargeted metabolomics datasets (e.g. 20 Orbitrap
  feature tables + MS2 spectra) must be annotated under competing knowledge-network
  topologies and compared using decoy-based accuracy, FDR, and same-superclass specificity
  metrics.
when_to_use_negative:
- The goal is single-layer annotation with data-driven molecular networking alone
  (e.g. GNPS FBMN/IIMN or NetID) rather than comparing knowledge-driven topology benchmarks.
- Input is already an annotation result table — this skill benchmarks annotation topologies
  from raw feature/MS2 inputs, not post-hoc re-scoring of existing identifications.
- The datasets lack decoy MS2 reference entries, making the decoy-based Top 3 accuracy
  and estimated FDR metrics uncomputable.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetDNA3
  role: Core two-layer interactive networking algorithm (MrnAnnoAlgo3) executing recursive
    annotation propagation under MRN/SMN/FCN topologies
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: MetDNA2
  role: Prior knowledge-guided multi-layer metabolic network workflow; its runMetDNA2()
    interface and dependency stack (igraph, dplyr, readr) illustrate the R processing
    environment
  repo: https://github.com/ZhuMetLab/MetDNA2
- name: igraph
  role: Builds the three knowledge-driven network topologies (MRN, SMN, FCN) used
    for propagation benchmarking
- name: readr
  role: Loads Supplementary Table 1 and Supplementary Data 4 decoy MS2 entries into
    R
- name: dplyr
  role: Aggregates accuracy, FDR, and specificity metrics across topologies into benchmark_topology_metrics.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:1c17521db1b08c3d96aa4a7f6198d59e019b43ea53fb5325463676ee972073d1
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# untargeted-metabolomics-dataset-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs recursive annotation propagation over knowledge-driven network topologies (MRN, SMN, FCN) on untargeted metabolomics datasets with decoy-MS2 benchmarking, to quantify how network connectivity affects annotation accuracy, estimated FDR, and propagation specificity.

## When to use

Use when a batch of untargeted metabolomics datasets (e.g. 20 Orbitrap feature tables + MS2 spectra) must be annotated under competing knowledge-network topologies and compared using decoy-based accuracy, FDR, and same-superclass specificity metrics.

## When NOT to use

- The goal is single-layer annotation with data-driven molecular networking alone (e.g. GNPS FBMN/IIMN or NetID) rather than comparing knowledge-driven topology benchmarks.
- Input is already an annotation result table — this skill benchmarks annotation topologies from raw feature/MS2 inputs, not post-hoc re-scoring of existing identifications.
- The datasets lack decoy MS2 reference entries, making the decoy-based Top 3 accuracy and estimated FDR metrics uncomputable.

## Inputs

- Supplementary Table 1 — list of 20 Orbitrap untargeted metabolomics datasets
- Supplementary Data 4 — decoy MS2 reference entries
- MS1 peak tables (.csv with 'name', 'mz', 'rt' columns) and MS2 data files (.mgf or .msp) per dataset
- Sample information table (.csv with 'sample.name' and 'group' columns)

## Outputs

- benchmark_topology_metrics.csv — summary table of same-superclass specificity, decoy Top 3 accuracy, and estimated FDR for MRN/SMN/FCN
- Per-dataset annotation result tables (annotation tables for base-peak candidates, peak groups, and feature-metabolite pairs)
- igraph network objects for the MRN, SMN, and FCN topologies

## How to apply

Load the dataset list (Supplementary Table 1, 20 Orbitrap untargeted metabolomics datasets) and the decoy MS2 reference entries (Supplementary Data 4) into R with readr. For each dataset, run the MrnAnnoAlgo3/MetDNA3 recursive annotation propagation separately under the three knowledge-driven topologies — MRN, SMN, FCN — building each network with igraph. For each topology, compute same-superclass propagation specificity (fraction of propagated annotation pairs sharing the same chemical superclass), decoy-based Top 3 accuracy (whether the true metabolite ranks within the top 3 candidates when ranked against decoy MS2 spectra), and the decoy-derived estimated FDR. Aggregate the three metrics for MRN/SMN/FCN into a summary table with dplyr (benchmark_topology_metrics.csv) and validate against reported values: specificity 80.8%/62.6%/24.9%, Top 3 accuracy 73.3%/65.1%/58.3%, FDR 15.8%/36.4%/44.5%. The rationale is that indiscriminately increasing network connectivity (MRN → SMN → FCN) degrades annotation quality, so the sparser MRN should outperform denser topologies.

## Related tools

- **MetDNA3** (Core two-layer interactive networking algorithm (MrnAnnoAlgo3) executing recursive annotation propagation under MRN/SMN/FCN topologies) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **MetDNA2** (Prior knowledge-guided multi-layer metabolic network workflow; its runMetDNA2() interface and dependency stack (igraph, dplyr, readr) illustrate the R processing environment) — https://github.com/ZhuMetLab/MetDNA2
- **igraph** (Builds the three knowledge-driven network topologies (MRN, SMN, FCN) used for propagation benchmarking)
- **readr** (Loads Supplementary Table 1 and Supplementary Data 4 decoy MS2 entries into R)
- **dplyr** (Aggregates accuracy, FDR, and specificity metrics across topologies into benchmark_topology_metrics.csv)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); runMetDNA2(path_pos="wd/POS", path_neg="wd/NEG", polarity="positive", instrument="orbitrap", column="hilic", ce="30")
```

## Evaluation signals

- Same-superclass propagation specificity matches 80.8% (MRN), 62.6% (SMN), 24.9% (FCN) within numerical tolerance
- Decoy-based Top 3 annotation accuracy matches 73.3% (MRN), 65.1% (SMN), 58.3% (FCN) within tolerance
- Estimated annotation FDR matches 15.8% (MRN), 36.4% (SMN), 44.5% (FCN) within tolerance
- Monotonic degradation order MRN > SMN > FCN holds across all three metrics — increasing connectivity must not improve annotation quality
- benchmark_topology_metrics.csv contains rows for all 20 Orbitrap datasets × 3 topologies with no missing metric values

## Limitations

- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so FDR estimates from MS2 matching may be optimistic.
- MetDNA2's spectral library objects (zhuMetLib, zhuMetlib_orbitrap, zhuRPlib, lib_rt, lib_ccs) are removed from the open-source package for copyright reasons — users must supply their own libraries and repackage.
- Cross-tool concordance may overestimate performance because chemical standards are lacking, and validation tools may have included validation metabolites in their training sets.
- No FDR is reported for the curated MRN itself due to the lack of clearly defined non-RPs (negative examples) and inapplicability of GNN performance metrics to the filtered candidate set.

## Evidence

- [results] increasing network connectivity from MRN to SMN to FCN degraded annotation accuracy (73.3% to 65.1% to 58.3%) and raised estimated FDR (15.8% to 36.4% to 44.5%): [evidence span withheld — anchor status: not_found]
- [results] the sparser MRN gave the highest same-superclass propagation specificity (80.8% vs 62.6% for SMN and 24.9% for FCN): [evidence span withheld — anchor status: not_found]
- [readme] MetDNA2 dependency stack showing the R processing environment with igraph, dplyr, readr: [evidence span withheld — anchor status: external_doc]
- [readme] MetDNA2 input format requirements for MS1 peak table and MS2 files: [evidence span withheld — anchor status: external_doc]
