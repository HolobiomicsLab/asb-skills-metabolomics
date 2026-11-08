---
name: operon-benchmarking-against-reference
description: Use when a whole-genome or metagenomic operon prediction run has produced
  a predicted-operon file (e.g. Operon_File.tab) and reference operon annotations
  exist, and you need sensitivity, specificity and accuracy to validate the pipeline.
when_to_use_negative:
- No reference operon annotation exists for the organism, so no TP/FP/FN ground truth
  can be constructed
- The goal is de novo operon discovery on novel metagenomic samples rather than validation
  against known operons
- Input is a gene-prediction .tab file with no predicted-operon file and you are still
  upstream of the operon-calling step
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaRon
  role: Produces the predicted operons (Operon_File.tab) via the 'ago' or 'op' process
    that are benchmarked against reference operons
  repo: https://github.com/zaidissa/MetaRon
- name: IDBA
  role: Assembles reads into scaftigs upstream of operon prediction in the 'ago' mode
    whose outputs are benchmarked
  repo: https://github.com/zaidissa/MetaRon
- name: prodigal
  role: Predicts genes on scaftigs whose co-directional proximal pairs are merged
    into the operons being evaluated
  repo: https://github.com/zaidissa/MetaRon
- name: NNPP
  role: De novo promoter prediction (2.2) whose evidence feeds the operon calls being
    scored
  repo: https://github.com/zaidissa/MetaRon
- name: NeSSM
  role: Simulates Illumina reads (two million reads at 15X) from the three-genome
    mixture used in the simulated-genome benchmark
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:f366d83581e37482925e9aefa27d0eaed34a4cb1d64f3a64df628692ac2d0753
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - NeSSM
  - Prodigal
  - Python
  - prodigal
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# operon-benchmarking-against-reference

## Summary

Benchmark predicted operons from a genome or metagenome run (e.g., MetaRon's Operon_File.tab) against curated reference operon annotations such as DOOR/RegulonDB by computing gene-pair-level sensitivity, specificity and accuracy, as done for E. coli K-12 MG1655 (97.8/94.1/92.4%), the simulated three-genome mixture (93.7/75.5/88.1%) and the E. coli c20 draft genome (87/91/88%).

## When to use

Use when a whole-genome or metagenomic operon prediction run has produced a predicted-operon file (e.g. Operon_File.tab) and reference operon annotations exist, and you need sensitivity, specificity and accuracy to validate the pipeline.

## When NOT to use

- No reference operon annotation exists for the organism, so no TP/FP/FN ground truth can be constructed
- The goal is de novo operon discovery on novel metagenomic samples rather than validation against known operons
- Input is a gene-prediction .tab file with no predicted-operon file and you are still upstream of the operon-calling step

## Inputs

- Predicted operons file (Operon_File.tab) from a MetaRon run
- Reference operon annotations for the organism (e.g. DOOR, ODB, RegulonDB)
- Reference genome coordinates (e.g. E. coli K-12 MG1655 NC_000913.3; E. coli c20 draft genome NGBR00000000.1)
- Gene prediction .tab file and scaftig FASTA when benchmarking the 'op' process

## Outputs

- Operon performance metrics table (e.g. operon_performance_metrics.csv or operon_performance_c20.csv) with sensitivity, specificity and accuracy
- Counts of true positives, false positives and false negatives at the gene-pair level

## How to apply

Collect predicted operons from the pipeline output (MetaRon writes Operon_File.tab from the 'ago' or 'op' process). Download or assemble a reference operon set for the organism(s) — e.g. DOOR operons for E. coli K-12 MG1655 (NC_000913.3), or the reference annotation of the E. coli c20 draft genome (GenBank NGBR00000000.1) — and map predicted operons onto the reference genome as gene-pair/coordinate sets. Match predicted to reference operons by coordinate overlap and count true positives, false positives and false negatives at the gene-pair level. Compute sensitivity as TP/(TP+FN), specificity as TN/(TN+FP), and accuracy as (TP+TN)/(TP+TN+FP+FN), writing the per-genome and pooled metrics to a CSV table (e.g. operon_performance_metrics.csv). The rationale is that coordinate-overlap matching at the gene-pair level decouples the benchmark from the specific genome assembly, allowing the same procedure for whole-genome runs, simulated mixtures and draft genomes.

## Related tools

- **MetaRon** (Produces the predicted operons (Operon_File.tab) via the 'ago' or 'op' process that are benchmarked against reference operons) — https://github.com/zaidissa/MetaRon
- **IDBA** (Assembles reads into scaftigs upstream of operon prediction in the 'ago' mode whose outputs are benchmarked) — https://github.com/zaidissa/MetaRon
- **prodigal** (Predicts genes on scaftigs whose co-directional proximal pairs are merged into the operons being evaluated) — https://github.com/zaidissa/MetaRon
- **NNPP** (De novo promoter prediction (2.2) whose evidence feeds the operon calls being scored) — https://github.com/zaidissa/MetaRon
- **NeSSM** (Simulates Illumina reads (two million reads at 15X) from the three-genome mixture used in the simulated-genome benchmark)

## Examples

```
metaron --sample SRP029211 --process ago --read_type merge --read_length r --paired_1 R1.fastq --paired_2 R2.fastq --output outdir/
```

## Evaluation signals

- Computed sensitivity, specificity and accuracy match the reported E. coli K-12 MG1655 whole-genome values of 97.8, 94.1 and 92.4% against DOOR
- Simulated three-genome mixture benchmark reproduces 93.7% sensitivity, 75.5% specificity and 88.1% accuracy within ~1 percentage point
- E. coli c20 draft genome benchmark reproduces 87, 91 and 88% sensitivity, specificity and accuracy
- Metrics table (CSV) contains explicit sensitivity, specificity and accuracy fields computed from TP/(TP+FN), TN/(TN+FP) and (TP+TN)/(TP+TN+FP+FN)
- For the MG1655 whole-genome run, intermediate scale should match the published run (82 scaftigs, 4227 genes, co-directional proximons with IGD < 601 bp) before scoring

## Limitations

- Specificity drops on the simulated metagenome (75.5% vs 94.1% on E. coli whole-genome), so benchmark results do not transfer across data types
- No wet-lab validation; benchmark agreement is computational only and deeper findings would require wet-lab resources
- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated into the pipeline, so downstream analysis must be performed separately on the predicted operons
- Pipeline restricted to Linux and academic use only

## Evidence

- [abstract] MetaRon consistently achieved high operon prediction sensitivity, specificity and accuracy across E. coli whole-genome (97.8, 94.1 and 92.4%), simulated genome (93.7, 75.5 and 88.1%) and E. coli c20: "MetaRon consistently achieved high operon prediction sensitivity, specificity and accuracy across E. coli whole-genome (97.8, 94.1 and 92.4%), simulated genome (93.7, 75.5 and 88.1%) and E. coli c20"
- [results] 82 scaftigs were assembled by MetaRon via IDBA [55].: "82 scaftigs were assembled by MetaRon via IDBA [55]."
- [results] MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes.: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [results] The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32].: "The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32]."
- [results] two million reads simulated at 15X depth via NeSSM (Next-Generation Simulator for Metagenomics) [66].: "two million reads simulated at 15X depth via NeSSM (Next-Generation Simulator for Metagenomics) [66]."
