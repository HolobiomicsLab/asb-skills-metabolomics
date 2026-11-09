---
name: decoy-spectra-fdr-estimation
description: Use when evaluating annotation accuracy and FDR of network-based metabolite
  annotation (e.g. MetDNA3 MRN/SMN/FCN propagation) against decoy MS2 reference spectra
  on Orbitrap untargeted metabolomics datasets.
when_to_use_negative:
- The goal is de novo annotation of unknown metabolites rather than benchmarking FDR/accuracy
  of an existing annotation run
- The data are not untargeted metabolomics MS2 datasets (e.g. targeted assays without
  MS2 spectra) since decoy-MS2 evaluation requires spectral matching
- No decoy MS2 reference spectra or negative-control spectral entries are available
  for the metabolite space being annotated
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetDNA3
  role: Performs the recursive annotation propagation whose candidate rankings are
    evaluated against decoy MS2 spectra
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: igraph
  role: Builds the MRN/SMN/FCN network topologies over which annotation propagation
    is run before decoy-based evaluation
- name: readr
  role: Loads Supplementary Table 1 (dataset list) and Supplementary Data 4 (decoy
    MS2 entries) into R
- name: dplyr
  role: Aggregates accuracy, FDR, and specificity metrics per topology into the benchmark
    summary table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:3690a4511ddef65622c556b0ef5b1f56d285a5a43bb07c0636f0cb68e30e678d
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
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# decoy-spectra-fdr-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking method that estimates the false discovery rate of network-based metabolite annotation (e.g. MetDNA3 propagation under MRN/SMN/FCN topologies) by ranking candidate annotations against decoy MS2 spectra and checking whether the true metabolite falls within the Top 3 candidates.

## When to use

Use when evaluating annotation accuracy and FDR of network-based metabolite annotation (e.g. MetDNA3 MRN/SMN/FCN propagation) against decoy MS2 reference spectra on Orbitrap untargeted metabolomics datasets.

## When NOT to use

- The goal is de novo annotation of unknown metabolites rather than benchmarking FDR/accuracy of an existing annotation run
- The data are not untargeted metabolomics MS2 datasets (e.g. targeted assays without MS2 spectra) since decoy-MS2 evaluation requires spectral matching
- No decoy MS2 reference spectra or negative-control spectral entries are available for the metabolite space being annotated

## Inputs

- Supplementary Table 1 — list of 20 Orbitrap untargeted metabolomics datasets
- Supplementary Data 4 — decoy MS2 reference entries
- Annotation candidate tables from MetDNA3/MrnAnnoAlgo3 propagation runs (MRN, SMN, FCN)
- Metabolite superclass annotations for specificity computation

## Outputs

- benchmark_topology_metrics.csv — per-topology decoy Top 3 accuracy, estimated FDR, and same-superclass propagation specificity (MRN/SMN/FCN)
- Per-dataset decoy-based ranking results of candidate annotations

## How to apply

Load the list of 20 Orbitrap untargeted metabolomics datasets (Supplementary Table 1) and the decoy MS2 reference entries (Supplementary Data 4) into R with readr. For each dataset, run the MrnAnnoAlgo3/MetDNA3 recursive annotation propagation under each knowledge-driven topology (MRN, SMN, FCN), building each network with igraph. Rank candidate annotations against the decoy MS2 spectra and score Top 3 accuracy by checking whether the true metabolite ranks within the top 3 candidates; derive the estimated annotation FDR from these decoy-based results. The rationale is that decoy spectra provide negative controls against which optimistic MS2-matching FDR estimates can be calibrated — the article warns that MS2-based validation alone may be optimistic because isomeric metabolites are indistinguishable by MS2 spectra alone. Aggregate Top 3 accuracy and estimated FDR per topology (expected benchmark values: accuracy 73.3%/65.1%/58.3% and FDR 15.8%/36.4%/44.5% for MRN/SMN/FCN) into a summary table with dplyr.

## Related tools

- **MetDNA3** (Performs the recursive annotation propagation whose candidate rankings are evaluated against decoy MS2 spectra) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **igraph** (Builds the MRN/SMN/FCN network topologies over which annotation propagation is run before decoy-based evaluation)
- **readr** (Loads Supplementary Table 1 (dataset list) and Supplementary Data 4 (decoy MS2 entries) into R)
- **dplyr** (Aggregates accuracy, FDR, and specificity metrics per topology into the benchmark summary table)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3"); benchmark <- read_csv("Supplementary_Table_1.csv") |> rowwise() |> mutate(top3_acc = decoy_top3_accuracy(anno), fdr = decoy_fdr(anno))
```

## Evaluation signals

- Computed decoy-based Top 3 accuracy matches reported benchmark values within numerical tolerance: 73.3% (MRN), 65.1% (SMN), 58.3% (FCN)
- Computed estimated FDR matches reported values: 15.8% (MRN), 36.4% (SMN), 44.5% (FCN)
- Same-superclass propagation specificity matches reported values: 80.8% (MRN), 62.6% (SMN), 24.9% (FCN)
- Metrics show the expected monotonic degradation as connectivity increases from MRN to SMN to FCN
- Output CSV contains one row per topology with all three metrics populated and consistent column schema

## Limitations

- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so FDR estimates from MS2 matching alone may be optimistic
- The estimated FDR depends on the quality and coverage of decoy MS2 reference entries in Supplementary Data 4
- Benchmark values are tied to the 20 Orbitrap datasets and may not transfer to other instruments or data-processing pipelines

## Evidence

- [results] Benchmark decoy-based Top 3 accuracy and FDR per topology: [evidence span withheld — anchor status: not_found]
- [methods] Decoy-MS2 evaluation setup: [evidence span withheld — anchor status: not_found]
