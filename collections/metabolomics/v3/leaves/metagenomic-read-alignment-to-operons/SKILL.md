---
name: metagenomic-read-alignment-to-operons
description: Use when you have predicted operon sequences from metagenomic shotgun
  samples and raw paired-end reads, and need per-sample pathway/KO abundances to compare
  disease versus control groups.
when_to_use_negative:
- You only need operon prediction itself and do not have raw reads to map back (use
  MetaRon's 'ago' or 'op' process instead)
- Your downstream goal is secondary metabolite identification from operonic sequences
  rather than pathway/KO differential abundance (use antiSMASH-based secondary metabolite
  analysis instead)
- Pre-computed gene/protein functional profiles are already available, making read-to-operon
  alignment and FMAP redundant
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: BOWTIE2
  role: Maps paired-end raw metagenomic reads to the predicted non-redundant operon
    sequences; installed via 'conda install -c bioconda bowtie'
  repo: https://github.com/zaidissa/MetaRon
- name: FMAP
  role: Functional Mapping and Analysis Pipeline that receives mapped reads, assigns
    KEGG KO identifiers, and supports differential abundance analysis of pathways
    between disease and control
- name: SAMtools
  role: Converts BOWTIE2 alignments to sorted BAM files
- name: MetaRon
  role: Predicts the non-redundant operon sequences used as the alignment reference
    for this skill
  repo: https://github.com/zaidissa/MetaRon
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:b97d943881f8de84c17c824f3d37f66a69ffa31fb8164e0334539eae69067f22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - BOWTIE2
  - FMAP
  - MetaRon
  - SAMtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metagenomic-read-alignment-to-operons

## Summary

Maps raw paired-end metagenomic reads back to predicted non-redundant operon sequences using BOWTIE/BOWTIE2 (with SAMtools for sorted BAM), then submits mapped reads to FMAP for KO assignment so operon-derived pathways can be quantified per sample and tested for differential abundance between conditions such as T2D versus control.

## When to use

Use when you have predicted operon sequences from metagenomic shotgun samples and raw paired-end reads, and need per-sample pathway/KO abundances to compare disease versus control groups.

## When NOT to use

- You only need operon prediction itself and do not have raw reads to map back (use MetaRon's 'ago' or 'op' process instead)
- Your downstream goal is secondary metabolite identification from operonic sequences rather than pathway/KO differential abundance (use antiSMASH-based secondary metabolite analysis instead)
- Pre-computed gene/protein functional profiles are already available, making read-to-operon alignment and FMAP redundant

## Inputs

- Non-redundant operon sequences predicted by MetaRon (Operon_File.tab-derived operonic sequences, e.g. 1,232,407 unique operons from 145 paired-end human gut samples)
- Paired-end raw metagenomic reads per sample (e.g. FASTQ files like ERR022075.1.fastq and ERR022075.2.fastq)
- Sample metadata partitioning samples into disease and control groups (e.g. 74 T2D vs 71 controls)

## Outputs

- Sorted BAM files of reads aligned to the operon reference (one per sample)
- FMAP-assigned KEGG KO identifiers per sample at sequence identity >= 80% and e-value >= 1e-10
- Per-sample KO abundance matrix
- Differential KO abundance table between disease and control groups (e.g. differential_ko_abundance.tsv), reporting associations such as K00691, K21279, K07101 with type 2 diabetes

## How to apply

Take the non-redundant operon sequences predicted by MetaRon (e.g. 1.23 million unique operons from 145 paired-end human gut metagenome samples) as the alignment reference. Map each sample's paired-end raw reads to the operon sequences using bowtie2 (install via 'conda install -c bioconda bowtie'), converting alignments to sorted BAM with SAMtools. Submit the mapped reads to the Functional Mapping and Analysis Pipeline (FMAP) to assign KEGG KO identifiers, applying sequence identity >= 80% and e-value >= 1e-10 thresholds. Build a per-sample KO abundance matrix from the FMAP output, partition samples into disease (e.g. 74 T2D) and control (e.g. 71 healthy) groups, and test differential abundance of each KO. This follows the README's downstream analysis for 'Operonnic pathways': mapping raw reads to operonic sequences with BOWTIE and submitting the mapped reads to FMAP.

## Related tools

- **BOWTIE2** (Maps paired-end raw metagenomic reads to the predicted non-redundant operon sequences; installed via 'conda install -c bioconda bowtie') — https://github.com/zaidissa/MetaRon
- **FMAP** (Functional Mapping and Analysis Pipeline that receives mapped reads, assigns KEGG KO identifiers, and supports differential abundance analysis of pathways between disease and control)
- **SAMtools** (Converts BOWTIE2 alignments to sorted BAM files)
- **MetaRon** (Predicts the non-redundant operon sequences used as the alignment reference for this skill) — https://github.com/zaidissa/MetaRon

## Examples

```
conda install -c bioconda bowtie; bowtie2 -x operons_index -1 ERR022075.1.fastq -2 ERR022075.2.fastq | samtools sort -o ERR022075_sorted.bam
```

## Evaluation signals

- Sorted BAM files exist for each of the 145 paired-end samples and are readable by SAMtools/BEDTools utilities
- FMAP output yields a KO abundance matrix with one column per sample and one row per KO, produced under identity >= 80% and e-value >= 1e-10 thresholds
- Differential KO table contains one row per KO and correctly reports strong T2D associations for Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279), and uncharacterized protein K07101
- Group sizes match the study partition: 74 T2D samples and 71 controls out of 145 total
- Sample counts and direction of association are consistent with the published finding of 1,232,407 unique operons from 145 paired-end human gut metagenome samples

## Limitations

- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated in the pipeline; the downstream pathway analysis was performed separately on the MetaRon predicted operons rather than within MetaRon
- Operonic clusters rely on co-directionality, intergenic distance (<= 600 bp IGD per README; < 601 bp in article) and promoter presence, so operons violating these criteria may be missed and reads on them unassignable
- Results are purely computational; no wet-lab validation was performed, so KO associations require further in-depth investigation
- Pipeline (including prerequisites like IDBA, prodigal, NNPP 2.2) is restricted to Linux and academic use only; MetaRon README requires Python 2.7

## Evidence

- [readme] downstream operonic pathways workflow: "a. Mapping raw metagenomic reads to operonic sequences using BOWTIE

	b. Submitting the mapped reads to Functional Mapping and Analysis Pipeline (FMAP)"
- [other] FMAP thresholds and workflow: [evidence span withheld — anchor status: not_found]
- [abstract] T2D KO association finding: "We also report strong association of type 2 diabetes with Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and an uncharacterized protein (K07101)."
- [abstract] operon reference dataset size: "Finally, we identified 1,232,407 unique operons from 145 paired-end human gut metagenome samples."
