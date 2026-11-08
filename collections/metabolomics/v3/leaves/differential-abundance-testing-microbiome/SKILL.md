---
name: differential-abundance-testing-microbiome
description: Use when a disease/control metagenome study requires testing which operon-derived
  KEGG ortholog pathways differ in abundance between groups, e.g. 74 T2D samples vs
  71 healthy controls from mapped gut metagenome reads.
when_to_use_negative:
- Input is already a KO/pathway abundance feature table from a different annotation
  source (no MetaRon operon reference or FMAP assignment), so the operon-to-KO mapping
  step is not applicable
- The dataset lacks a two-condition contrast (e.g. single-cohort or time-series-only
  metagenomes), which the disease-vs-control differential test assumes
- Reads cannot be mapped to the predicted operons (e.g. unrelated environments), making
  the bowtie2-to-operon reference strategy invalid
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: MetaRon
  role: Provides the non-redundant predicted operon sequences used as the mapping
    reference for downstream abundance analysis
  repo: https://github.com/zaidissa/MetaRon
- name: BOWTIE2
  role: Maps paired-end raw reads of each sample to the operon sequences before functional
    assignment (installed via 'conda install -c bioconda bowtie')
- name: FMAP
  role: Functional Mapping and Analysis Pipeline that assigns KEGG KO identifiers
    to mapped reads for the differential abundance test
- name: SAMtools
  role: Converts bowtie2 alignments to sorted BAM files as input to FMAP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:22a52e13e6b549ec9561d414c0b24018c8cb76bf638050da2d10b1bc6305daab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - BOWTIE2
  - FMAP
  - MetaRon
  - SAMtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# differential-abundance-testing-microbiome

## Summary

Performs disease-versus-control differential abundance analysis of functional orthologs (KEGG KOs) assigned to MetaRon-predicted metagenomic operons, using read mapping (bowtie2) and FMAP functional annotation across 145 human gut metagenome samples (74 T2D vs 71 controls).

## When to use

Use when a disease/control metagenome study requires testing which operon-derived KEGG ortholog pathways differ in abundance between groups, e.g. 74 T2D samples vs 71 healthy controls from mapped gut metagenome reads.

## When NOT to use

- Input is already a KO/pathway abundance feature table from a different annotation source (no MetaRon operon reference or FMAP assignment), so the operon-to-KO mapping step is not applicable
- The dataset lacks a two-condition contrast (e.g. single-cohort or time-series-only metagenomes), which the disease-vs-control differential test assumes
- Reads cannot be mapped to the predicted operons (e.g. unrelated environments), making the bowtie2-to-operon reference strategy invalid

## Inputs

- Non-redundant MetaRon-predicted operon sequences (1,232,407 unique operons) as alignment reference
- Paired-end raw metagenome reads (FASTQ) for 145 human gut samples
- Sample metadata partitioning 74 T2D disease samples vs 71 healthy controls
- FMAP-assigned KO annotations per mapped read

## Outputs

- Sorted BAM files of reads mapped to operon sequences (via SAMtools)
- Per-sample KEGG KO abundance matrix derived from FMAP output
- Differential KO table saved as differential_ko_abundance.tsv with one row per KO, direction and significance of T2D associations

## How to apply

Take the non-redundant operon sequences predicted by MetaRon (1.23 million unique operons from 145 paired-end human gut metagenome samples) as the alignment reference. Map each sample's paired-end raw reads to the operons with bowtie2 (installed via 'conda install -c bioconda bowtie') and convert alignments to sorted BAM with SAMtools. Submit mapped reads to FMAP to assign KEGG KO identifiers under the study's thresholds of sequence identity >= 80% and e-value >= 1e-10, then build a per-sample KO abundance matrix. Partition samples into disease (74 T2D) and control (71) groups, test each KO for differential abundance, and record direction and significance. The rationale is that operons act as a subset representing secondary-metabolite/pathway trends in whole-metagenome data, so group-level KO shifts reveal disease-associated operonic pathways.

## Related tools

- **MetaRon** (Provides the non-redundant predicted operon sequences used as the mapping reference for downstream abundance analysis) — https://github.com/zaidissa/MetaRon
- **BOWTIE2** (Maps paired-end raw reads of each sample to the operon sequences before functional assignment (installed via 'conda install -c bioconda bowtie'))
- **FMAP** (Functional Mapping and Analysis Pipeline that assigns KEGG KO identifiers to mapped reads for the differential abundance test)
- **SAMtools** (Converts bowtie2 alignments to sorted BAM files as input to FMAP)

## Examples

```
bowtie2 -x metaron_operons -1 sample_1.fastq -2 sample_2.fastq | samtools sort -o sample.sorted.bam
```

## Evaluation signals

- differential_ko_abundance.tsv exists and has one row per KO
- The table correctly reports strong T2D associations for Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and the uncharacterized protein K07101, as stated in the paper
- Group sizes match the study design: 74 T2D samples and 71 controls partitioned from the 145 samples
- FMAP KO assignments respect the study thresholds: sequence identity >= 80% and e-value >= 1e-10
- KO abundance matrix contains a column per sample covering all 145 paired-end human gut metagenome samples

## Limitations

- Secondary metabolite prediction, pathway annotation and graphical representation were not yet integrated into the MetaRon pipeline (planned future work); the analysis was performed separately on predicted operons rather than integrated
- MetaRon is restricted to Linux and academic use only
- Findings are computational only — no wet-lab validation was performed, so associations would require further in-depth investigation with wet-lab resources

## Evidence

- [abstract] Strong T2D KO associations: "We also report strong association of type 2 diabetes with Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and an uncharacterized protein (K07101)."
- [results] Operon set size: "Finally, we identified 1,232,407 unique operons from 145 paired-end human gut metagenome samples."
- [readme] FMAP downstream analysis: "b. Submitting the mapped reads to Functional Mapping and Analysis Pipeline (FMAP)"
- [other] Differential abundance test step: [evidence span withheld — anchor status: not_found]
- [abstract] Operons as metabolite trends: "We also demonstrate the use of operons as a subset to represent the trends of secondary metabolites in whole-metagenome data"
