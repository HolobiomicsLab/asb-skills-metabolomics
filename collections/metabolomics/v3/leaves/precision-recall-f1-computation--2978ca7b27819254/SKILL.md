---
name: precision-recall-f1-computation
description: Use when a mock-community or simulated metagenomic benchmark provides
  known ground-truth abundances and you must score inferred gene-cluster abundance
  calls across a MASH-threshold × Bowtie2-mode grid.
when_to_use_negative:
- No ground-truth or mock-community abundances exist (e.g., real oral/IBD metagenome
  samples) — differential abundance testing is the appropriate alternative, not F1
  scoring
- Input is already a finished performance table of precision/recall values — only
  selection/visualization of the best cell remains
- The comparison targets raw read-mapping quality rather than downstream abundance
  recapitulation
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: BiG-MAP
  role: 'Pipeline under benchmark: bigmap-family performs MASH redundancy filtering
    per threshold, bigmap-map maps reads (Bowtie2 modes), bigmap-analyse quantifies
    abundances used for the F1 grid'
  repo: github.com/medema-group/BiG-MAP
- name: Grinder
  role: Simulates the eight mock-community metagenomic libraries that supply the ground-truth
    abundances for precision/recall/F1
- name: Bowtie2
  role: Aligns simulated reads to representative gene clusters in end-to-end and local
    modes (incl. fast preset) to form the mode axis of the grid
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:7c29f43c375d8c72de33544e95e278631009d18624fa172f758b750e511cfbba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - Bowtie2
  - Grinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# precision-recall-f1-computation

## Summary

Benchmark skill for scoring BiG-MAP gene-cluster abundance calls against known ground-truth abundances from simulated metagenomic libraries, computing precision, recall, and F1 across a MASH-similarity × Bowtie2-mode parameter grid to select pipeline defaults.

## When to use

Use when a mock-community or simulated metagenomic benchmark provides known ground-truth abundances and you must score inferred gene-cluster abundance calls across a MASH-threshold × Bowtie2-mode grid.

## When NOT to use

- No ground-truth or mock-community abundances exist (e.g., real oral/IBD metagenome samples) — differential abundance testing is the appropriate alternative, not F1 scoring
- Input is already a finished performance table of precision/recall values — only selection/visualization of the best cell remains
- The comparison targets raw read-mapping quality rather than downstream abundance recapitulation

## Inputs

- Simulated metagenomic FASTQ libraries with known ground-truth gene-cluster abundances (e.g., Grinder simulations at 0.5× and 0.05× coverage)
- Reference gene-cluster nucleotide sequences from antiSMASH/gutSMASH output folders
- Per-cell quantified gene-cluster abundance tables produced by bigmap-analyse
- Grid specification of MASH similarity thresholds (-tg) and Bowtie2 alignment modes (end-to-end vs. local, incl. fast preset)

## Outputs

- Precision, recall, and F1 values (each in [0,1]) for every MASH-threshold × Bowtie2-mode combination
- Heatmap-style results table across the parameter grid
- Recorded best-performing grid cell used to justify pipeline defaults

## How to apply

Simulate reference-community metagenomes (in the BiG-MAP paper: eight libraries from 101 CGR reference genomes via Grinder at 0.5× and 0.05× coverage with uniform, linear, power-law, and exponential abundance structures) so ground-truth abundances are known. For each grid cell, first run the MinHash/MASH redundancy filtering in bigmap-family at the candidate similarity threshold (default -tg 0.8), then map simulated reads to the non-redundant representatives with bigmap-map, trying Bowtie2 in both end-to-end and local modes including the fast preset. Quantify abundances with bigmap-analyse, binarize or compare against the Grinder ground truth to derive true/false positives, and compute precision = TP/(TP+FP), recall = TP/(TP+FN), F1 = 2·P·R/(P+R) per cell. Assemble the metrics into a heatmap-style table and record the best-performing grid cell; the paper's optimum was MASH similarity 0.8 with Bowtie2 fast end-to-end alignment, which became the tool's defaults.

## Related tools

- **BiG-MAP** (Pipeline under benchmark: bigmap-family performs MASH redundancy filtering per threshold, bigmap-map maps reads (Bowtie2 modes), bigmap-analyse quantifies abundances used for the F1 grid) — github.com/medema-group/BiG-MAP
- **Grinder** (Simulates the eight mock-community metagenomic libraries that supply the ground-truth abundances for precision/recall/F1)
- **Bowtie2** (Aligns simulated reads to representative gene clusters in end-to-end and local modes (incl. fast preset) to form the mode axis of the grid)

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/ -tg 0.8
```

## Evaluation signals

- All precision, recall, and F1 values fall within [0,1]
- The optimal grid cell matches the reported defaults: MASH similarity cutoff 0.8 and Bowtie2 fast end-to-end mode
- Every MASH-threshold × Bowtie2-mode combination yields a filled metric cell before ranking
- F1 values are consistent with the identity F1 = 2·P·R/(P+R) given the tabulated precision and recall
- Abundance calls compared against the Grinder ground truth reproduce the paper's conclusion that MGC abundances are accurately recapitulated with simulated data

## Limitations

- Benchmark validity is tied to Grinder-simulated libraries (0.5× and 0.05× coverage, four abundance structures); the winning mode/cutoff may not transfer to real shotgun metagenomes
- MASH filtering alone is approximate; the BiG-MAP README strongly recommends a second BiG-SCAPE redundancy-filtering round for more accurate GCF grouping
- The BiG-MAP tool has only been tested on Ubuntu per the README
- F1 depends on how abundance calls are binarized relative to ground truth, so thresholds for calling a cluster 'detected' must be fixed before comparing grid cells

## Evidence

- [intro] Validation with simulated data showing accurate recapitulation: "We validated the tool using simulated metagenomic data and show how MGC abundance and expression levels are accurately recapitulated."
- [intro] MinHash redundancy filtering rationale and BiG-SCAPE grouping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] MASH default 0.8 similarity cutoff is configurable per grid cell: [evidence span withheld — anchor status: external_doc]
- [readme] bigmap-family module groups clusters into GCFs, prerequisite to mapping: "Group gene clusters onto gene cluster families (GCFs) and housekeeping gene
   families (HGFs) using bigmap-family"
