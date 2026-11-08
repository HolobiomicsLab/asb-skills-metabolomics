---
name: computation-runtime-benchmarking
description: Use when comparing wall-clock runtimes of alternative NetGSA computational
  variants (NetGSA.REHE.Cl, NetGSA.REHE, NetGSA.REML.All) on cancer gene expression
  datasets, or when validating that an overhauled pipeline meets a runtime budget
  (minutes on a personal computer).
when_to_use_negative:
- The analysis goal is statistical interpretation of enriched pathways rather than
  measuring or comparing computational efficiency of NetGSA variants
- Only one computational variant will be run — with no REML baseline or no-clustering
  comparison there is nothing to benchmark against
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: R package providing prepareAdjMat (network estimation with optional pathway
    clustering) and NetGSA (pathway enrichment with lklMethod='REHE' or 'REML'), the
    two stages whose combined runtime is benchmarked
  repo: github.com/mikehellstern/netgsa
- name: knitr
  role: R markdown vignette engine used to document and reproduce the runtime benchmark
    workflow
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:273946364c44aab1d17f79ff0c2db60b32cdf72c68d48b96c6235a568d606abf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - Reactome
  - knitr
  - netgsa
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# computation-runtime-benchmarking

## Summary

Benchmarking the combined runtime of network estimation (prepareAdjMat) and topology-based pathway enrichment (NetGSA) under different computational variants — REHE vs REML variance component estimation, with and without pathway clustering — to quantify efficiency gains without sacrificing statistical power.

## When to use

Use when comparing wall-clock runtimes of alternative NetGSA computational variants (NetGSA.REHE.Cl, NetGSA.REHE, NetGSA.REML.All) on cancer gene expression datasets, or when validating that an overhauled pipeline meets a runtime budget (minutes on a personal computer).

## When NOT to use

- The analysis goal is statistical interpretation of enriched pathways rather than measuring or comparing computational efficiency of NetGSA variants
- Only one computational variant will be run — with no REML baseline or no-clustering comparison there is nothing to benchmark against

## Inputs

- Gene expression matrices (breast cancer 2012 dataset: 520 subjects, 2784 genes; TCGA prostate cancer dataset from GEO/TCGA accession in ref [24])
- Curated pathway interaction data from KEGG and Reactome databases
- netgsa R package with prepareAdjMat and NetGSA functions

## Outputs

- Aggregated runtime comparison table (mean and standard deviation per computational variant per dataset, e.g. table1_runtimes.csv)
- End-to-end runtime measurement and pathway results table (pathway_tests_results.csv) with test statistics, p-values, and FDR q-values for the 114 KEGG pathways

## How to apply

Load the gene expression datasets (breast cancer 2012 with 520 subjects / 2784 genes; TCGA prostate) together with curated pathway interaction data in R. Estimate pathway adjacency matrices with prepareAdjMat (use cluster=TRUE for the clustered variant, since clustering genes within pathways is what drives the large combined-runtime reduction). Run NetGSA under three timed variants — NetGSA.REHE.Cl (REHE with pathway clustering), NetGSA.REHE (no clustering), and NetGSA.REML.All — timing each with R's system-level timing (system.time) and repeating to accumulate stable measurements. Aggregate repeated timings into mean and standard deviation per variant per dataset and write the comparison table (e.g. table1_runtimes.csv). For the end-to-end efficiency check, run the full workflow with lklMethod='REHE' and sampling=TRUE (sample_n=0.25, sample_p=0.25) and confirm the pipeline completes on the order of minutes (~2 minutes reported on a 2-CPU machine with 16 GB RAM).

## Related tools

- **netgsa** (R package providing prepareAdjMat (network estimation with optional pathway clustering) and NetGSA (pathway enrichment with lklMethod='REHE' or 'REML'), the two stages whose combined runtime is benchmarked) — github.com/mikehellstern/netgsa
- **knitr** (R markdown vignette engine used to document and reproduce the runtime benchmark workflow)

## Examples

```
runtime <- system.time(res <- NetGSA(x, groups, pathways, lklMethod="REHE", sampling=TRUE, sample_n=0.25, sample_p=0.25))
```

## Evaluation signals

- The timing table must show NetGSA.REHE several times faster than NetGSA.REML.All, with an additional large runtime reduction from clustering (NetGSA.REHE.Cl fastest)
- The full REHE + sampling pipeline (sample_n=0.25, sample_p=0.25) should complete on the order of minutes — consistent with the reported ~2 minutes on a 2-CPU personal computer with 16 GB RAM
- Each variant should be timed repeatedly to accumulate stable measurements before aggregating mean and standard deviation
- The results table must contain p-value and FDR columns for all tested pathways (all 114 KEGG pathways), confirming no statistical power was sacrificed for speed

## Limitations

- Runtime gains depend on REHE variance component estimation and pathway clustering; REML on the entire network is markedly slower
- The NetGSA implementation removed the compatibility check between adjacency matrices and input data for block-diagonal structures across groups, and the authors note filtering input variables to those belonging to at least one tested pathway before running NetGSA
- README flags open questions (e.g. whether beta output is needed given the block diagonal structure of D), indicating the optimized implementation is still under active revision

## Evidence

- [abstract] abstract: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [intro] intro: "To overcome these limitations, this paper provides important updates to an existing statistically powerful method, NetGSA [3, 4], which was previously difficult to use and computationally expensive"
