---
name: mash-similarity-threshold-optimization
description: Use when building a BiG-MAP-style abundance-profiling workflow where
  redundant gene clusters (e.
when_to_use_negative:
- The gene cluster reference set is already curated and non-redundant, so no MinHash
  filtering cutoff needs to be chosen
- No ground-truth (simulated) abundances are available, so F1 against known composition
  cannot be computed — defaults must be taken on faith instead of tuned
- The goal is functional pathway profiling from databases like KEGG/MetaCyc rather
  than mapping reads to gene cluster sequences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BiG-MAP
  role: 'Pipeline under tuning: bigmap-family performs the MASH MinHash redundancy
    filtering at the candidate thresholds; bigmap-map (Bowtie2) and bigmap-analyse
    produce the abundance calls scored by F1'
  repo: github.com/medema-group/BiG-MAP
- name: Grinder
  role: Simulates metagenomic libraries with known abundance structures (uniform,
    linear, power-law, exponential) to provide ground truth for precision/recall/F1
- name: Bowtie2
  role: Aligner whose end-to-end vs local modes (fast preset) form the second dimension
    of the optimization grid
- name: gutSMASH
  role: Predicts the metabolic gene clusters that feed the redundancy-filtering input
    directories
  repo: victoriapascal/gutsmash
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:8f659eabab876456b486d19abc792fc9485525590027bfe010bac318c587e2e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - Bowtie2
  - Grinder
  - gutSMASH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mash-similarity-threshold-optimization

## Summary

Tunes the MASH MinHash similarity cutoff used for gene-cluster redundancy filtering in BiG-MAP by grid-searching thresholds against known (simulated) abundances and selecting the cutoff that maximizes precision/recall F1. BiG-MAP's own benchmark found MASH similarity 0.8 with Bowtie2 'fast' end-to-end alignment to be optimal, so 0.8 is the tool default.

## When to use

Use when building a BiG-MAP-style abundance-profiling workflow where redundant gene clusters (e.g. antiSMASH/gutSMASH output from many genomes) cause ambiguous read mapping, and you need evidence for (or must tune) the MinHash similarity threshold via simulated libraries with ground-truth abundances.

## When NOT to use

- The gene cluster reference set is already curated and non-redundant, so no MinHash filtering cutoff needs to be chosen
- No ground-truth (simulated) abundances are available, so F1 against known composition cannot be computed — defaults must be taken on faith instead of tuned
- The goal is functional pathway profiling from databases like KEGG/MetaCyc rather than mapping reads to gene cluster sequences

## Inputs

- antiSMASH or gutSMASH output directories containing predicted gene clusters per genome
- Reference genomes with known gene cluster inventories (101 CGR reference genomes in the benchmark)
- Simulated metagenomic read libraries in FASTQ format with known ground-truth abundances (e.g. Grinder output at 0.5x and 0.05x coverage)

## Outputs

- Non-redundant representative gene cluster reference set per threshold (BiG-MAP.GCF.fna / BiG-MAP.GCF.bed / BiG-MAP.GCs.json)
- Per-sample gene cluster abundance table from bigmap-analyse
- Grid of precision/recall/F1 values across MASH-threshold x Bowtie2-mode combinations (heatmap-style results table) and the best-performing cell

## How to apply

Simulate metagenomic libraries from the reference genomes with known abundance structures (in the BiG-MAP paper, eight libraries from 101 CGR reference genomes were simulated with Grinder at 0.5x and 0.05x coverage under uniform, linear, power-law, and exponential abundance profiles). For each candidate MASH similarity cutoff in the grid (fraction 0-1, default 0.8; exposed as the -tg/-th flags of bigmap-family), run the MinHash-based redundancy filtering step to produce non-redundant representative gene clusters. Map each simulated library's reads to the representatives with bigmap-map, sweeping Bowtie2 end-to-end vs local modes (including the 'fast' preset), quantify abundances with bigmap-analyse, and compute precision, recall, and F1 against the Grinder ground truth for every threshold-x-mode cell. Select the grid cell with the highest F1; in the paper this was MASH similarity 0.8 with Bowtie2 'fast' end-to-end mode, which became the BiG-MAP defaults.

## Related tools

- **BiG-MAP** (Pipeline under tuning: bigmap-family performs the MASH MinHash redundancy filtering at the candidate thresholds; bigmap-map (Bowtie2) and bigmap-analyse produce the abundance calls scored by F1) — github.com/medema-group/BiG-MAP
- **Grinder** (Simulates metagenomic libraries with known abundance structures (uniform, linear, power-law, exponential) to provide ground truth for precision/recall/F1)
- **Bowtie2** (Aligner whose end-to-end vs local modes (fast preset) form the second dimension of the optimization grid)
- **gutSMASH** (Predicts the metabolic gene clusters that feed the redundancy-filtering input directories) — victoriapascal/gutsmash

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -tg 0.8 -O /usr001/results_family/
```

## Evaluation signals

- All computed precision, recall, and F1 values fall within [0, 1] for every MASH-threshold x Bowtie2-mode grid cell
- The optimal grid cell reproduces the published default: MASH similarity 0.8 combined with Bowtie2 'fast' end-to-end alignment mode
- bigmap-family outputs the expected artifacts (BiG-MAP.GCF.fna, BiG-MAP.GCF.bed, BiG-MAP.GCs.json) for each threshold tested
- Abundances quantified by bigmap-analyse on simulated libraries correlate with the Grinder ground-truth abundance structure
- The similarity threshold remains a fraction between 0 and 1 (0 = every cluster its own family, 1 = one family containing all clusters)

## Limitations

- The 0.8 default was calibrated on simulated mock-community data from 101 CGR reference genomes; other reference sets or coverage regimes (e.g. far below 0.05x) may favor different cutoffs
- The first MASH filtering step is coarse — the README strongly recommends adding the second BiG-SCAPE redundancy-filtering round (with a processed Pfam-A.hmm via hmmpress) for more accurate family assignment
- MinHash filtering exists specifically to avoid ambiguous mapping; too permissive a threshold keeps redundant clusters that mis-assign reads, while too strict a threshold over-collapses distinct clusters
- BiG-MAP has only been tested on Ubuntu

## Evidence

- [intro] BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] Default MASH cutoff and -tg flag: [evidence span withheld — anchor status: external_doc]
- [readme] -tg semantics bound the search space: [evidence span withheld — anchor status: external_doc]
- [intro] Validation rationale with simulated data: "We validated the tool using simulated metagenomic data and show how MGC abundance and expression levels are accurately recapitulated."
- [readme] bigmap-family CLI usage: [evidence span withheld — anchor status: external_doc]
