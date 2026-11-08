---
name: metagenomic-short-read-assembly
description: Use when the input is unassembled paired-end shotgun FASTQ reads (e.g.
  Illumina HiSeq from SRA studies such as SRP029211 or SRP008047) and scaftigs must
  be generated before gene and operon prediction.
when_to_use_negative:
- Scaffolds and gene predictions already exist — run MetaRon in 'op' mode (--igp gene_prediction.tab
  --isc scaftigs.fa) instead of re-assembling
- Reads are longer than 128 bases in a single paired file and read_type/read_length
  flags are not adjusted, which misdirects the IDBA stage
- The goal is transcriptomic or amplicon analysis rather than shotgun whole-genome/metagenome
  operon prediction
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: IDBA
  role: De Bruijn graph de novo assembler that converts MetaRon input FASTQ reads
    into scaftigs (82 scaftigs on the E. coli K-12 MG1655 benchmark); installed via
    conda install -c bioconda idba
  repo: https://github.com/zaidissa/MetaRon
- name: MetaRon
  role: Pipeline wrapper that invokes IDBA within the 'ago' process and applies the
    ≤ 500 bp scaftig length filter before gene prediction
  repo: https://github.com/zaidissa/MetaRon
- name: Prodigal
  role: Predicts genes (4,227 genes on the E. coli run) on the assembled and length-filtered
    scaftigs, verifying the assembly is usable
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:028f1a5b791f8fb62eb56a87b1abf9d2f838c73af08b24114100e88eb6047c62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - Prodigal
  - Python
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metagenomic-short-read-assembly

## Summary

De novo assembly of paired-end shotgun metagenomic or whole-genome reads into scaftigs with IDBA, as the first stage of the MetaRon operon-prediction pipeline, followed by removal of scaftigs ≤ 500 bp to prepare a reliable scaffold set for downstream gene prediction.

## When to use

Use when the input is unassembled paired-end shotgun FASTQ reads (e.g. Illumina HiSeq from SRA studies such as SRP029211 or SRP008047) and scaftigs must be generated before gene and operon prediction.

## When NOT to use

- Scaffolds and gene predictions already exist — run MetaRon in 'op' mode (--igp gene_prediction.tab --isc scaftigs.fa) instead of re-assembling
- Reads are longer than 128 bases in a single paired file and read_type/read_length flags are not adjusted, which misdirects the IDBA stage
- The goal is transcriptomic or amplicon analysis rather than shotgun whole-genome/metagenome operon prediction

## Inputs

- Paired-end Illumina HiSeq FASTQ read files (R1/R2 or merged single file), e.g. from SRA run SRP029211 or the 145 human gut samples of SRP008047
- MetaRon config.txt with the NNPP2.2 executable path registered for the full 'ago' pipeline

## Outputs

- Assembled scaftigs (FASTA) from IDBA
- Filtered scaftig set with scaffolds ≤ 500 bp removed
- Downstream Prodigal/MetaGeneMark gene predictions and Operon_File.tab operon catalogue

## How to apply

Run IDBA de novo assembly inside MetaRon's 'ago' process (metaron --sample <name> --process ago --read_type merge|paired --read_length r|l --paired_1 R1.fastq --paired_2 R2.fastq --output <dir>), choosing 'merge' when paired reads sit in two files and 'l' or 'r' depending on whether reads are longer or shorter than 128 bases. The assembly converts raw reads into scaftigs; on the E. coli K-12 MG1655 HiSeq benchmark this produced 82 scaftigs. Filter out scaftigs with length ≤ 500 bp, since only sufficiently long scaftigs support reliable co-directional proximon and operon calling downstream. Pass the filtered scaftig set to Prodigal or MetaGeneMark gene prediction, which on the E. coli run yielded 4,227 genes, then to the IGD < 601 bp proximon/operon stage. Sanity-check the intermediate scale (e.g. 82 scaftigs, 4,227 genes) against published values to confirm the assembly stage behaved correctly.

## Related tools

- **IDBA** (De Bruijn graph de novo assembler that converts MetaRon input FASTQ reads into scaftigs (82 scaftigs on the E. coli K-12 MG1655 benchmark); installed via conda install -c bioconda idba) — https://github.com/zaidissa/MetaRon
- **MetaRon** (Pipeline wrapper that invokes IDBA within the 'ago' process and applies the ≤ 500 bp scaftig length filter before gene prediction) — https://github.com/zaidissa/MetaRon
- **Prodigal** (Predicts genes (4,227 genes on the E. coli run) on the assembled and length-filtered scaftigs, verifying the assembly is usable)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output ~/path/to/output/directory/
```

## Evaluation signals

- Assembly stage reproduces the published intermediate scale for the E. coli K-12 MG1655 benchmark: 82 scaftigs assembled via IDBA
- Gene prediction on filtered scaftigs yields approximately 4,227 genes with Prodigal on the same benchmark
- No scaftig ≤ 500 bp remains in the scaffold set passed to gene prediction
- Downstream operon metrics against DOOR reach 97.8% sensitivity, 94.1% specificity and 92.4% accuracy for the whole-genome E. coli run
- Output directory contains the expected Operon_File.tab with operon records derived from the assembled scaftigs

## Limitations

- Pipeline is restricted to Linux and academic use only
- Very short reads (< 128 bases) require the 'r' read_length flag; using the wrong flag misdirects assembly
- Assembly quality depends on scaftig length — scaftigs ≤ 500 bp are discarded, so poorly assembled regions contribute no genes or operons

## Evidence

- [results] 82 scaftigs were assembled by MetaRon via IDBA: "82 scaftigs were assembled by MetaRon via IDBA"
- [results] Scaftigs ≤ 500 bp removal: "Scaftigs with length less than or equal to 500 bp were removed."
- [results] Prodigal gene prediction on scaftigs: "The remaining scaftigs resulted in 4227 genes, predicted using prodigal"
- [intro] Pipeline stages including de novo assembly: "performs necessary downstream data processing (de novo assembly, gene prediction, de novo promoter prediction and proximon prediction), before identifying the operons"
- [readme] README assembly within 'ago' process: "assembly into scaffolds via IDBA, data manipulation, gene prediction via prodigal and lastly operon prediction based on gene's co-directionality, intergenic distance (IGD) and promoters"
