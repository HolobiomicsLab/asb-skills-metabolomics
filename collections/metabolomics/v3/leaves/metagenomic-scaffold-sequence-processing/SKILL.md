---
name: metagenomic-scaffold-sequence-processing
description: Use when input is metagenomic shotgun data or pre-assembled scaftigs
  with gene predictions (.tab from Prodigal or MetaGeneMark) and the goal is de novo
  operon prediction without functional or experimental information.
when_to_use_negative:
- The goal requires functional annotation or experimental evidence — MetaRon deliberately
  predicts operons without any functional or experimental data
- Input is already an operon set or the analysis is downstream secondary-metabolite/pathway
  analysis, which the README treats as separate (antiSMASH/BOWTIE) work
- Running outside Linux or in a non-academic setting — the article notes the pipeline
  is restricted to Linux and academic use only
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaRon
  role: Command-line pipeline orchestrating scaftig processing, gene prediction, promoter
    prediction and operon prediction (processes 'ago' and 'op')
  repo: https://github.com/zaidissa/MetaRon
- name: IDBA
  role: De novo assembly of filtered raw reads into scaftigs prior to gene and operon
    prediction
- name: prodigal
  role: Gene prediction on scaftigs, selected with --tool 2 in the 'op' process
- name: NNPP
  role: Neural Network Promoter Prediction 2.2 used via promoter_prediction() on up
    to 700 bp upstream sequences of proximon genes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:eb91a44ed38dd6d9083460760e5e3bcc480ee18942951b5071267bc6321e6115
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - prodigal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metagenomic-scaffold-sequence-processing

## Summary

A MetaRon-based workflow that processes metagenomic scaftigs (from IDBA assembly or pre-assembled input) through gene prediction with Prodigal or MetaGeneMark, co-directional proximon clustering (IGD < 601 bp), NNPP 2.2 promoter prediction, and Prom_IGD_Clustering to delimit operons without functional or experimental data.

## When to use

Use when input is metagenomic shotgun data or pre-assembled scaftigs with gene predictions (.tab from Prodigal or MetaGeneMark) and the goal is de novo operon prediction without functional or experimental information.

## When NOT to use

- The goal requires functional annotation or experimental evidence — MetaRon deliberately predicts operons without any functional or experimental data
- Input is already an operon set or the analysis is downstream secondary-metabolite/pathway analysis, which the README treats as separate (antiSMASH/BOWTIE) work
- Running outside Linux or in a non-academic setting — the article notes the pipeline is restricted to Linux and academic use only

## Inputs

- Filtered paired-end metagenomic raw reads (FASTQ, e.g. ERR022075.1.fastq / ERR022075.2.fastq) for process 'ago'
- Pre-assembled scaftigs FASTA file (--isc) for process 'op'
- Gene prediction .tab file generated with MetaGeneMark or Prodigal (--igp)
- NNPP2.2 executable path recorded in config.txt

## Outputs

- Scaftigs assembled by IDBA (e.g. 82 scaftigs for E. coli K-12 MG1655 HiSeq reads)
- Predicted genes (4227 genes via Prodigal in the E. coli K-12 test case)
- Co-directional proximon clusters (IGD < 601 bp; 822 clusters with 2955 genes in the E. coli K-12 run)
- Tab-delimited Operon_File.tab with predicted operon records

## How to apply

Run MetaRon with --process 'ago' to assemble filtered raw reads via IDBA, or --process 'op' when scaftigs and gene predictions already exist. Remove scaftigs of length <= 500 bp, then predict genes using Prodigal (--tool 2) or MetaGeneMark (--tool 1) from the scaftig FASTA (--isc) and gene .tab file (--igp). Identify co-directional proximal gene clusters (proximons) using the intergenic distance criterion IGD < 601 bp. Extract up to 700 bp upstream sequences and run NNPP 2.2 via promoter_prediction(), adding the NNPP2.2 path to config.txt first; integrate promoter predictions with the proximon table via Prom_IGD_Clustering(), which uses promoter presence/absence upstream and downstream of genes to define transcription unit boundaries. Write the final operons to the tab-delimited Operon_File.tab in the specified output directory.

## Related tools

- **MetaRon** (Command-line pipeline orchestrating scaftig processing, gene prediction, promoter prediction and operon prediction (processes 'ago' and 'op')) — https://github.com/zaidissa/MetaRon
- **IDBA** (De novo assembly of filtered raw reads into scaftigs prior to gene and operon prediction)
- **prodigal** (Gene prediction on scaftigs, selected with --tool 2 in the 'op' process)
- **NNPP** (Neural Network Promoter Prediction 2.2 used via promoter_prediction() on up to 700 bp upstream sequences of proximon genes)

## Examples

```
metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 2 --output ~/path/to/output/directory/
```

## Evaluation signals

- Operon_File.tab exists in the output directory, is tab-delimited, and its records conform to the scaftig and gene prediction file format documented for the 'op' process
- Scaftigs <= 500 bp have been removed before gene prediction
- Proximon clusters satisfy the IGD < 601 bp co-directionality criterion (benchmark: 822 clusters / 2955 genes on E. coli K-12 MG1655)
- Operon boundaries are consistent with promoter presence/absence upstream and downstream of genes as predicted by NNPP 2.2
- Sensitivity/specificity/accuracy should fall in the reported ranges (97.8/94.1/92.4% on E. coli whole-genome; 93.7/75.5/88.1% on simulated genome; 87/91/88% on E. coli c20)

## Limitations

- NNPP can fail to identify promoters, which limits boundary definition in those cases
- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated into the pipeline (planned future work)
- Pipeline is restricted to Linux and academic use only; no wet-lab validation of predictions
- Requires Python 2.7 per the README prerequisites, while the article lists Python > 3.0 — environment compatibility must be checked

## Evidence

- [intro] The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene: "The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene"
- [results] MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes.: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [results] Scaftigs with length less than or equal to 500 bp were removed.: "Scaftigs with length less than or equal to 500 bp were removed."
- [results] The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32].: "The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32]."
- [readme] Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp.: "Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp."
- [readme] This will save metagenomic operon predictions ``Operon_File.tab``.: "This will save metagenomic operon predictions ``Operon_File.tab``."
