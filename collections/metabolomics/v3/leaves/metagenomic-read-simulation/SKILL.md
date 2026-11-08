---
name: metagenomic-read-simulation
description: 'Use when benchmarking a metagenomic abundance profiler (e.g. BiG-MAP)
  against known truth: you need simulated shotgun libraries with known gene-cluster
  abundances to sweep parameter grids (MASH cutoff × Bowtie2 mode) and compute F1.'
when_to_use_negative:
- You need to profile abundance in real (non-simulated) microbiome samples — simulation
  only validates recapitulation of known abundances, not real-world biological signal
- Input is already read data from a real sequencing run, where ground-truth gene cluster
  abundances do not exist to score against
- The tool under test does not map reads to gene cluster sequences, so a simulation
  ground-truth F1 comparison is not meaningful
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Grinder
  role: Simulates the metagenomic libraries from 101 CGR reference genomes at 0.5x/0.05x
    coverage under uniform, linear, power-law, and exponential abundance structures,
    providing ground-truth abundances
- name: BiG-MAP
  role: 'Tool under benchmark: bigmap-family (MASH redundancy filtering across thresholds),
    bigmap-map (Bowtie2 read mapping), and bigmap-analyse (abundance quantification)
    produce the calls that are scored'
  repo: github.com/medema-group/BiG-MAP
- name: Bowtie2
  role: Aligns simulated reads to representative gene clusters in both end-to-end
    and local modes (including the fast preset) for each point of the parameter grid
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:e2c8e971d15656a58911fad14361881ab8abd0ed888f95946b4bbe41d6d840d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - Bowtie2
  - Grinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metagenomic-read-simulation

## Summary

Simulate metagenomic shotgun libraries with Grinder from reference genomes at known abundance structures to create ground-truth data for benchmarking gene-cluster abundance profilers such as BiG-MAP, and score profiler calls with precision/recall/F1.

## When to use

Use when benchmarking a metagenomic abundance profiler (e.g. BiG-MAP) against known truth: you need simulated shotgun libraries with known gene-cluster abundances to sweep parameter grids (MASH cutoff × Bowtie2 mode) and compute F1.

## When NOT to use

- You need to profile abundance in real (non-simulated) microbiome samples — simulation only validates recapitulation of known abundances, not real-world biological signal
- Input is already read data from a real sequencing run, where ground-truth gene cluster abundances do not exist to score against
- The tool under test does not map reads to gene cluster sequences, so a simulation ground-truth F1 comparison is not meaningful

## Inputs

- 101 CGR (complete genome) reference genome sequences
- Predicted gene cluster output folders from antiSMASH/gutSMASH runs
- Grinder simulation parameters: coverage levels (0.5x, 0.05x) and abundance profiles (uniform, linear, power-law, exponential)
- Grinder ground-truth per-cluster abundances

## Outputs

- Eight simulated metagenomic FASTQ libraries
- Per-sample gene cluster abundance calls from bigmap-analyse
- Precision, recall, and F1 per MASH-threshold x Bowtie2-mode grid cell
- Heatmap-style results table with the best-performing grid cell identified

## How to apply

Simulate eight metagenomic libraries from the 101 CGR reference genomes using Grinder at 0.5x and 0.05x coverage, spanning uniform, linear, power-law, and exponential abundance structures, and retain the Grinder ground-truth abundances per gene cluster. Feed the predicted gene cluster folders through the BiG-MAP family step (bigmap-family), sweeping the MASH similarity threshold across the grid (default 0.8) to obtain non-redundant representative clusters. Map each simulated library to the representatives with bigmap-map, running Bowtie2 in both end-to-end and local modes (including the fast preset) for every MASH-threshold x mode combination. Quantify gene cluster abundance per library with bigmap-analyse, then compute precision, recall, and F1 against the Grinder ground truth for every grid cell and assemble a heatmap-style results table. Select the best-performing cell as the tool defaults; in this benchmark that was MASH similarity 0.8 with Bowtie2 'fast' end-to-end alignment.

## Related tools

- **Grinder** (Simulates the metagenomic libraries from 101 CGR reference genomes at 0.5x/0.05x coverage under uniform, linear, power-law, and exponential abundance structures, providing ground-truth abundances)
- **BiG-MAP** (Tool under benchmark: bigmap-family (MASH redundancy filtering across thresholds), bigmap-map (Bowtie2 read mapping), and bigmap-analyse (abundance quantification) produce the calls that are scored) — github.com/medema-group/BiG-MAP
- **Bowtie2** (Aligns simulated reads to representative gene clusters in both end-to-end and local modes (including the fast preset) for each point of the parameter grid)

## Evaluation signals

- All computed precision, recall, and F1 values fall in [0,1]
- The optimal grid cell matches the reported best defaults: MASH similarity cutoff 0.8 and Bowtie2 'fast' end-to-end alignment mode
- Each simulated library was generated from one of the specified coverage (0.5x, 0.05x) and abundance profile (uniform, linear, power-law, exponential) combinations
- The full grid covers every MASH-threshold x Bowtie2-mode combination across all eight libraries, and metrics are assembled into a complete heatmap-style table
- bigmap-analyse abundance calls for a given cluster are consistent with the Grinder ground-truth abundance of the genome it was simulated from

## Limitations

- Simulation validates recapitulation of known abundances only; performance on real, complex microbiome communities (e.g. the 96 oral caries samples) is assessed separately
- F1 results depend on the chosen abundance structures and coverage levels; conclusions may not generalize to other distributions or depths
- BiG-MAP has only been tested on Ubuntu, per the repository README

## Evidence

- [intro] Simulated-data validation of BiG-MAP: "We validated the tool using simulated metagenomic data and show how MGC abundance and expression levels are accurately recapitulated."
- [readme] MASH cutoff default used in the family step: [evidence span withheld — anchor status: external_doc]
- [intro] Reads are mapped to reference gene cluster sequences: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
