---
name: kegg-ortholog-functional-annotation
description: Use when raw paired-end metagenome reads must be converted into per-sample
  KEGG ortholog (KO) abundance profiles using predicted operon sequences as the alignment
  reference, e.g. for disease-vs-control pathway association testing.
when_to_use_negative:
- When functional annotation from already-predicted genes is available and read-mapping-based
  KO assignment is unnecessary (MetaRon's 'op' mode operates directly from scaftigs
  and gene predictions)
- When the goal is secondary metabolite analysis of operons — the README routes that
  to antiSMASH, not FMAP
- When reads cannot be assumed to share sequence identity >= 80% with the operon reference,
  since the study's FMAP thresholds depend on high-identity mapping
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3697
tools:
- name: MetaRon
  role: Provides the non-redundant operon sequences used as the bowtie2 alignment
    reference for KO annotation
  repo: https://github.com/zaidissa/MetaRon
- name: BOWTIE2
  role: Maps each sample's paired-end raw reads to the predicted operon sequences;
    installed via 'conda install -c bioconda bowtie'
  repo: https://github.com/zaidissa/MetaRon
- name: FMAP
  role: Functional Mapping and Analysis Pipeline that assigns KEGG KO identifiers
    to the mapped reads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:687c78a3855d696051702195406eff197121a6a90569f993d980eaaf7186ff52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - BOWTIE2
  - FMAP
  - MetaRon
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# kegg-ortholog-functional-annotation

## Summary

Assign KEGG KO identifiers to metagenomic operon-derived function by mapping raw paired-end reads back to MetaRon-predicted non-redundant operon sequences with bowtie2 and submitting the mapped reads to the Functional Mapping and Analysis Pipeline (FMAP), producing a per-sample KO abundance matrix for downstream differential analysis.

## When to use

Use when raw paired-end metagenome reads must be converted into per-sample KEGG ortholog (KO) abundance profiles using predicted operon sequences as the alignment reference, e.g. for disease-vs-control pathway association testing.

## When NOT to use

- When functional annotation from already-predicted genes is available and read-mapping-based KO assignment is unnecessary (MetaRon's 'op' mode operates directly from scaftigs and gene predictions)
- When the goal is secondary metabolite analysis of operons — the README routes that to antiSMASH, not FMAP
- When reads cannot be assumed to share sequence identity >= 80% with the operon reference, since the study's FMAP thresholds depend on high-identity mapping

## Inputs

- Paired-end raw metagenome reads (FASTQ, e.g. 145 human gut samples)
- Non-redundant operon sequences predicted by MetaRon (reference for bowtie2 index)

## Outputs

- Sorted BAM files of reads mapped to operon sequences (via SAMtools)
- FMAP KO assignment output with sequence identity >= 80% and e-value >= 1e-10
- Per-sample KO abundance matrix (rows = KOs, columns = samples)
- Differential KO table, e.g. differential_ko_abundance.tsv, reporting direction and significance of T2D associations

## How to apply

First obtain the non-redundant operon set predicted by MetaRon (the paper removed operonic redundancy to yield 1.23 million unique operons from 145 paired-end human gut metagenome samples) and use it as the alignment reference. Map each sample's paired-end raw reads to the operon sequences with bowtie2 (installed via 'conda install -c bioconda bowtie'), then convert alignments to sorted BAM with SAMtools. Submit the mapped reads to FMAP to assign KEGG KO identifiers, applying the study's thresholds of sequence identity >= 80% and e-value >= 1e-10. Build a per-sample KO abundance matrix from the FMAP output and partition samples into groups (in the paper: 74 T2D samples vs 71 healthy controls) before testing differential abundance of each KO. This workflow reflects the README's proposed downstream analysis of 'Operonnic pathways' (BOWTIE mapping, then FMAP), and in the paper it surfaced strong type 2 diabetes associations with K00691 (Maltose phosphorylase), K21279 (3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase) and K07101 (uncharacterized protein).

## Related tools

- **MetaRon** (Provides the non-redundant operon sequences used as the bowtie2 alignment reference for KO annotation) — https://github.com/zaidissa/MetaRon
- **BOWTIE2** (Maps each sample's paired-end raw reads to the predicted operon sequences; installed via 'conda install -c bioconda bowtie') — https://github.com/zaidissa/MetaRon
- **FMAP** (Functional Mapping and Analysis Pipeline that assigns KEGG KO identifiers to the mapped reads)

## Examples

```
bowtie2 -x metaron_operons -1 ERR022075_1.fastq -2 ERR022075_2.fastq | samtools sort -o ERR022075.bam; fmap_submit ERR022075.bam --identity 80 --evalue 1e-10
```

## Evaluation signals

- The per-sample KO abundance matrix has one column per sample (74 T2D + 71 controls = 145 samples) and one row per KO
- The differential KO table (differential_ko_abundance.tsv) exists and correctly reports strong T2D associations for K00691 (Maltose phosphorylase), K21279 (3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase) and K07101 (uncharacterized protein)
- FMAP assignments respect the study thresholds: sequence identity >= 80% and e-value >= 1e-10
- Every sample produced a sorted BAM of reads mapped against the ~1.23 million non-redundant operon reference
- bowtie2 alignments are properly converted to coordinate-sorted BAM with SAMtools before FMAP submission

## Limitations

- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated into the MetaRon pipeline itself; 'in the future we plan to integrate the prediction of secondary metabolites, pathway annotation and graphical representation within the pipeline'
- 'In this research, the analysis was performed separately on the MetaRon predicted operons', so the FMAP/BOWTIE step is external downstream analysis rather than part of the pipeline
- Associations are computational only: 'more in-depth investigation, aided with wet-lab resources, could provide insightful findings'

## Evidence

- [abstract] Finally, we identified 1,232,407 unique operons from 145 paired-end human gut metagenome samples.: "Finally, we identified 1,232,407 unique operons from 145 paired-end human gut metagenome samples."
- [abstract] T2D association of three KOs: "We also report strong association of type 2 diabetes with Maltose phosphorylase (K00691), 3-deoxy-D-glycero-D-galacto-nononate 9-phosphate synthase (K21279) and an uncharacterized protein (K07101)."
- [readme] README downstream analysis: "a. Mapping raw metagenomic reads to operonic sequences using BOWTIE
	
	b. Submitting the mapped reads to Functional Mapping and Analysis Pipeline (FMAP)"
- [readme] README BOWTIE prerequisite: "BOWTIE (Optional: only required for downstream analysis)"
- [discussion] Analysis performed outside pipeline: "In this research, the analysis was performed separately on the MetaRon predicted operons"
