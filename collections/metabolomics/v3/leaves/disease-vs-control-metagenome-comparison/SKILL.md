---
name: disease-vs-control-metagenome-comparison
description: Use when paired-end gut metagenome reads must be mapped to predicted
  operon sequences and per-sample KO/pathway differential abundance tested between
  disease and control cohorts.
when_to_use_negative:
- Input is already a KO/pathway abundance table with no raw reads to map to operons
  — go directly to differential testing
- The metagenomic data lack predicted operons and no MetaRon reference has been produced
  — run operon prediction first
- The goal is metagenomic operon prediction itself rather than disease-vs-control
  abundance comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaRon
  role: Predicts the non-redundant operon sequences used as the mapping reference
    for the disease-vs-control comparison
  repo: https://github.com/zaidissa/MetaRon
- name: BOWTIE2
  role: Maps paired-end raw reads of each sample to the operon reference sequences
    (installed via conda bioconda)
- name: FMAP
  role: Functional Mapping and Analysis Pipeline that assigns KEGG KO identifiers
    to mapped reads and supports differential abundance analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:e0f6ddb67d42807c38c74cd0320c18d6d6bfbb92aa80fffa310c61f63c19997b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - BOWTIE2
  - FMAP
  - MetaRon
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# disease-vs-control-metagenome-comparison

## Summary

Compares KEGG KO pathway abundances between a disease group (74 type 2 diabetes samples) and a control group (71 samples) from 145 paired-end human gut metagenome samples, surfacing disease-associated pathways such as Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and uncharacterized protein K07101.

## When to use

Use when paired-end gut metagenome reads must be mapped to predicted operon sequences and per-sample KO/pathway differential abundance tested between disease and control cohorts.

## When NOT to use

- Input is already a KO/pathway abundance table with no raw reads to map to operons — go directly to differential testing
- The metagenomic data lack predicted operons and no MetaRon reference has been produced — run operon prediction first
- The goal is metagenomic operon prediction itself rather than disease-vs-control abundance comparison

## Inputs

- Non-redundant MetaRon-predicted operon sequences from 145 paired-end human gut metagenome samples (reference FASTA)
- Paired-end raw metagenome reads per sample (FASTQ)
- Sample metadata assigning 74 T2D samples and 71 control samples

## Outputs

- Per-sample KO abundance matrix
- Differential KO table (differential_ko_abundance.tsv) with one row per KO, direction and significance of T2D-vs-control association

## How to apply

Take the non-redundant operon sequences predicted by MetaRon (1.23 million unique operons from the 145 human gut samples) as the alignment reference and map each sample's paired-end raw reads with bowtie2 (installed via 'conda install -c bioconda bowtie'), converting alignments to sorted BAM with SAMtools. Submit the mapped reads to the Functional Mapping and Analysis Pipeline (FMAP) to assign KEGG KO identifiers using the study thresholds of sequence identity >= 80% and e-value >= 1e-10. Build a per-sample KO abundance matrix from the FMAP output, partition samples into 74 T2D versus 71 control samples, and test differential abundance of each KO between groups, recording direction and significance. The rationale is that operons act as a subset representing secondary metabolite/pathway trends in whole-metagenome data, so differential abundance at the operon-derived KO level links metagenomic structure to disease phenotype.

## Related tools

- **MetaRon** (Predicts the non-redundant operon sequences used as the mapping reference for the disease-vs-control comparison) — https://github.com/zaidissa/MetaRon
- **BOWTIE2** (Maps paired-end raw reads of each sample to the operon reference sequences (installed via conda bioconda))
- **FMAP** (Functional Mapping and Analysis Pipeline that assigns KEGG KO identifiers to mapped reads and supports differential abundance analysis)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ERR022075.1.fastq --paired_2 ERR022075.2.fastq --output ./out/
```

## Evaluation signals

- differential_ko_abundance.tsv exists with one row per KO
- Output reports strong type 2 diabetes associations for Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and uncharacterized protein K07101
- FMAP assignments respect thresholds of sequence identity >= 80% and e-value >= 1e-10
- Cohort partition is correct: 74 T2D samples and 71 controls out of 145 samples
- BAM files are coordinate-sorted and every sample's reads mapped against the 1.23 million unique operon reference

## Limitations

- Secondary metabolite prediction, pathway annotation and graphical representation were not integrated into the pipeline (planned future work); the analysis was performed separately on predicted operons
- No wet-lab validation was performed; findings require in-depth investigation aided with wet-lab resources
- Pipeline is restricted to Linux and academic use only

## Evidence

- [abstract] T2D association of K00691, K21279, K07101 via FMAP differential abundance: "We also report strong association of type 2 diabetes with Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and an uncharacterized protein (K07101)."
- [abstract] 1.23M unique operons from 145 gut samples: "Finally, we identified 1,232,407 unique operons from 145 paired-end human gut metagenome samples."
- [readme] README downstream analysis: bowtie mapping + FMAP: "a. Mapping raw metagenomic reads to operonic sequences using BOWTIE
	
	b. Submitting the mapped reads to Functional Mapping and Analysis Pipeline (FMAP)
	
	c. Identifying differential abundance of"
