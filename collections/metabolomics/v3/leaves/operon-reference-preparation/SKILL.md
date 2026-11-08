---
name: operon-reference-preparation
description: Use when raw shotgun metagenome reads (paired-end FASTQ) must be quantified
  against predicted operon sequences to generate per-sample pathway/KO abundance profiles
  for disease-vs-control comparisons.
when_to_use_negative:
- Input is already a gene/KO abundance table or pre-computed pathway profile — mapping
  reads to operons would be redundant.
- The metagenome lacks predicted operons (no MetaRon output or pre-assembled scaftigs
  + gene predictions available for the 'op' process).
- Reads are single-end or long-read data, which the BOWTIE2 paired-end mapping workflow
  described here does not cover.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaRon
  role: Predicts the metagenomic operons whose sequences serve as the alignment reference
    for read mapping
  repo: https://github.com/zaidissa/MetaRon
- name: BOWTIE2
  role: Maps each sample's paired-end raw reads onto the non-redundant operon sequences
    before FMAP submission
  repo: https://github.com/zaidissa/MetaRon
- name: FMAP
  role: Receives the mapped reads and assigns KEGG KO identifiers for pathway differential
    abundance analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:8733d8429c886c4738ece5a21a20cdbc521d3a23be42ce9c50b995d901ed59fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - BOWTIE2
  - FMAP
  - MetaRon
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# operon-reference-preparation

## Summary

Prepares a non-redundant operon sequence reference from MetaRon predictions and maps raw paired-end metagenomic reads against it with BOWTIE2 so mapped reads can be submitted to FMAP for KO/pathway abundance analysis. This bridges operon prediction and downstream differential pathway analysis.

## When to use

Use when raw shotgun metagenome reads (paired-end FASTQ) must be quantified against predicted operon sequences to generate per-sample pathway/KO abundance profiles for disease-vs-control comparisons.

## When NOT to use

- Input is already a gene/KO abundance table or pre-computed pathway profile — mapping reads to operons would be redundant.
- The metagenome lacks predicted operons (no MetaRon output or pre-assembled scaftigs + gene predictions available for the 'op' process).
- Reads are single-end or long-read data, which the BOWTIE2 paired-end mapping workflow described here does not cover.

## Inputs

- Non-redundant operon sequences predicted by MetaRon (e.g. from Operon_File.tab / scaftig-derived operon FASTA)
- Paired-end raw shotgun metagenome reads in FASTQ format (two files per sample, e.g. ERR022075.1.fastq and ERR022075.2.fastq)
- Sample metadata labeling disease (e.g. T2D) vs control groups

## Outputs

- Sorted BAM files of reads mapped to operon references (one per sample, via SAMtools)
- Per-sample KEGG KO abundance matrix from FMAP output
- Differential KO/pathway abundance table separating disease vs control samples (e.g. differential_ko_abundance.tsv)

## How to apply

Take the operon predictions emitted by MetaRon (Operon_File.tab and the underlying operonic sequences) as the alignment reference. Map each sample's paired-end raw reads (FASTQ) to the operon sequences using BOWTIE2 (installed via 'conda install -c bioconda bowtie'), and convert alignments to sorted BAM with SAMtools. Submit the mapped reads to the Functional Mapping and Analysis Pipeline (FMAP) to assign KEGG KO identifiers, applying thresholds of sequence identity >= 80% and e-value >= 1e-10 as used in the T2D analysis. Build a per-sample KO abundance matrix from the FMAP output, then partition samples into groups (e.g. 74 T2D vs 71 healthy controls). Remove operonic redundancy before mapping so that the reference contains only unique operons — the paper's 145 human gut samples yielded 1,232,407 unique operons after redundancy removal.

## Related tools

- **MetaRon** (Predicts the metagenomic operons whose sequences serve as the alignment reference for read mapping) — https://github.com/zaidissa/MetaRon
- **BOWTIE2** (Maps each sample's paired-end raw reads onto the non-redundant operon sequences before FMAP submission) — https://github.com/zaidissa/MetaRon
- **FMAP** (Receives the mapped reads and assigns KEGG KO identifiers for pathway differential abundance analysis)

## Examples

```
conda install -c bioconda bowtie && bowtie2 -x operon_reference -1 sample_1.fastq -2 sample_2.fastq | samtools sort -o sample_operons.bam
```

## Evaluation signals

- After redundancy removal the reference contains only unique operons (in the paper's run: 1,232,407 unique operons from 145 paired-end human gut samples).
- Each sample produces a valid sorted BAM from bowtie2/SAMtools with non-zero mapped reads.
- FMAP output yields a per-sample KO abundance matrix with one row per KO and columns for all 145 samples (or the study's cohort).
- The differential table correctly reproduces strong T2D associations for K00691 (Maltose phosphorylase), K21279 (3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase) and K07101 (uncharacterized protein).
- Group sizes match the study design: 74 T2D samples and 71 controls.

## Limitations

- MetaRon and its downstream analysis are restricted to Linux and academic use only.
- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated into the pipeline; the analysis was performed separately on predicted operons.
- BOWTIE and antiSMASH are optional README dependencies required only for the proposed downstream analysis, not the core operon prediction.

## Evidence

- [readme] Mapping raw metagenomic reads to operonic sequences using BOWTIE: "a. Mapping raw metagenomic reads to operonic sequences using BOWTIE"
- [readme] FMAP submission for differential pathway abundance: "b. Submitting the mapped reads to Functional Mapping and Analysis Pipeline (FMAP)"
- [abstract] 1.23 million unique operons from 145 human gut samples: "Finally, we identified 1,232,407 unique operons from 145 paired-end human gut metagenome samples."
- [abstract] Strong T2D associations for K00691, K21279, K07101: "We also report strong association of type 2 diabetes with Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and an uncharacterized protein (K07101)."
- [discussion] Analysis performed separately on predicted operons: "In this research, the analysis was performed separately on the MetaRon predicted operons"
