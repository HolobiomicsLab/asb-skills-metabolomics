---
name: bowtie2-alignment-mode-comparison
description: Use when selecting or validating Bowtie2 alignment mode/preset for metagenomic
  read mapping to gene cluster references, or when reproducibly benchmarking abundance-profiling
  accuracy against simulated libraries with known abundances.
when_to_use_negative:
- When only real (non-simulated) metagenomes are available and no ground-truth abundances
  exist — F1 against truth cannot be computed; use differential-abundance analysis
  instead
- When the goal is GCF grouping rather than mapping-parameter selection — use the
  BiG-SCAPE redundancy-filtering step of bigmap-family, not alignment-mode tuning
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: BiG-MAP
  role: 'Pipeline under test: bigmap-family (MASH redundancy filtering), bigmap-map
    (Bowtie2 read mapping), bigmap-analyse (abundance quantification)'
  repo: https://github.com/medema-group/BiG-MAP
- name: Grinder
  role: Simulates metagenomic libraries with known abundance structures to provide
    ground truth for F1 scoring
- name: Bowtie2
  role: Aligner whose end-to-end vs. local modes and presets are compared; 'fast'
    end-to-end was optimal
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:a7a35e687eaa1c2f2b605157431b464c2ac2b61e5339f12fec2ac32ea51a618b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - Bowtie2
  - Grinder
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bowtie2-alignment-mode-comparison

## Summary

Benchmark Bowtie2 end-to-end vs. local alignment modes (including the 'fast' preset) by mapping reads to non-redundant representative gene clusters and scoring abundance calls against known ground truth with precision/recall/F1. In the BiG-MAP mock-community benchmark, MASH similarity 0.8 plus Bowtie2 'fast' end-to-end mode yielded the best F1 and became the tool defaults.

## When to use

Use when selecting or validating Bowtie2 alignment mode/preset for metagenomic read mapping to gene cluster references, or when reproducibly benchmarking abundance-profiling accuracy against simulated libraries with known abundances.

## When NOT to use

- When only real (non-simulated) metagenomes are available and no ground-truth abundances exist — F1 against truth cannot be computed; use differential-abundance analysis instead
- When the goal is GCF grouping rather than mapping-parameter selection — use the BiG-SCAPE redundancy-filtering step of bigmap-family, not alignment-mode tuning

## Inputs

- Simulated shotgun metagenomic FASTQ libraries with known per-genome abundance profiles (e.g., Grinder output from 101 CGR reference genomes)
- Gene cluster nucleotide reference sequences (e.g., antiSMASH/gutSMASH output folders for each genome)
- Ground-truth per-cluster abundance table from the simulation
- Grid specification of MASH similarity cutoffs and Bowtie2 modes (end-to-end vs. local, fast preset)

## Outputs

- Precision, recall, and F1 per MASH-cutoff x Bowtie2-mode combination
- Heatmap-style results table of the parameter grid with the best-performing cell highlighted
- Per-library gene cluster abundance profiles from bigmap-analyse for each tested mode

## How to apply

Simulate metagenomic libraries (e.g., with Grinder at 0.5x and 0.05x coverage under uniform, linear, power-law, and exponential abundance profiles) so ground-truth gene cluster abundances are known. First reduce reference redundancy with the BiG-MAP family step, which uses MASH redundancy filtering at a sequence-similarity cutoff (grid over thresholds; 0.8 is the default) to avoid ambiguous mapping, producing BiG-MAP.GCF.fna as the mapping reference. For each simulated library, run bigmap-map, which maps reads to the representative gene clusters with Bowtie2 in both end-to-end and local modes (including the 'fast' end-to-end preset), then quantify per-cluster abundance with bigmap-analyse. Compute precision, recall, and F1 by comparing BiG-MAP abundance calls to the Grinder ground truth for every MASH-cutoff x Bowtie2-mode combination, assemble the grid into a heatmap-style table, and select the best-performing cell (F1 in [0,1]; expected optimum: MASH 0.8, Bowtie2 fast end-to-end).

## Related tools

- **BiG-MAP** (Pipeline under test: bigmap-family (MASH redundancy filtering), bigmap-map (Bowtie2 read mapping), bigmap-analyse (abundance quantification)) — https://github.com/medema-group/BiG-MAP
- **Grinder** (Simulates metagenomic libraries with known abundance structures to provide ground truth for F1 scoring)
- **Bowtie2** (Aligner whose end-to-end vs. local modes and presets are compared; 'fast' end-to-end was optimal)

## Examples

```
bowtie2 --fast --end-to-end -x BiG-MAP.GCF -1 sample_R1.fastq -2 sample_R2.fastq -S sample.sam
```

## Evaluation signals

- All computed precision, recall, and F1 values fall within [0,1]
- The optimal grid cell matches the reported defaults: MASH similarity cutoff 0.8 and Bowtie2 'fast' end-to-end alignment mode
- Each simulated library (0.5x and 0.05x coverage; uniform, linear, power-law, exponential profiles) yields a filled grid cell for every MASH-threshold x Bowtie2-mode combination
- bigmap-family output contains the expected artifacts (BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json) used as the mapping reference
- Abundance calls from bigmap-analyse reproduce the direction and rank of Grinder ground-truth abundances for the winning parameter set

## Limitations

- The benchmark relies on Grinder-simulated libraries; performance on real metagenomes with unknown true abundances is not directly measured by this skill
- BiG-MAP has been 'only tested on Ubuntu', so reproducing the grid may require that environment
- MASH filtering alone can leave ambiguous mappings; the authors strongly recommend the optional second redundancy-filtering round with BiG-SCAPE (which requires a processed Pfam-A.hmm and hmmpress)

## Evidence

- [readme] The first redundancy filtering step is performed by MASH, that by default uses 0.8 sequence similarity cut-off but can be changed as desired.: [evidence span withheld — anchor status: external_doc]
- [readme] BiG-MAP.GCF.fna = Reference file to map the WGS reads to: [evidence span withheld — anchor status: external_doc]
- [intro] Validation with simulated metagenomic data: "We validated the tool using simulated metagenomic data and show how MGC abundance and expression levels are accurately recapitulated."
- [intro] Mapping reads to gene cluster sequences: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
- [readme] bigmap-family input requirement: [evidence span withheld — anchor status: external_doc]
