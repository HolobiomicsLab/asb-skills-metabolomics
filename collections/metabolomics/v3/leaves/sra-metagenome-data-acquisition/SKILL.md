---
name: sra-metagenome-data-acquisition
description: Use when a cohort's raw shotgun metagenomic or metatranscriptomic reads
  must be retrieved from NCBI SRA via a BioProject accession list (e.g. PRJNA389280)
  before mapping reads to gene clusters.
when_to_use_negative:
- The reads are already available locally as FASTQ files — `bigmap-download` is optional
  and only needed when data still reside in SRA.
- The target data are not present in the SRA database — the article notes bigmap-download
  fetches shotgun data 'if present in the SRA database'.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP (bigmap-download module)
  role: Entry point that takes the accession list and downloads plus converts the
    cohort's metagenomic/metatranscriptomic reads from SRA
  repo: github.com/medema-group/BiG-MAP
- name: ncbi/sra-tools
  role: Provides fastq-dump, used to convert downloaded .SRA files into .fastq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:ef31503a63a5886f1901916244388c871f5dcf63e43da54edd0c76ed39d6dbd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-MAP (bigmap-download module)
  - ncbi/sra-tools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# sra-metagenome-data-acquisition

## Summary

Fetches raw shotgun metagenomic and metatranscriptomic reads from the NCBI Sequence Read Archive using the BiG-MAP `bigmap-download` module, converting SRA archives to FASTQ for downstream MGC abundance/expression mapping.

## When to use

Use when a cohort's raw shotgun metagenomic or metatranscriptomic reads must be retrieved from NCBI SRA via a BioProject accession list (e.g. PRJNA389280) before mapping reads to gene clusters.

## When NOT to use

- The reads are already available locally as FASTQ files — `bigmap-download` is optional and only needed when data still reside in SRA.
- The target data are not present in the SRA database — the article notes bigmap-download fetches shotgun data 'if present in the SRA database'.

## Inputs

- SRA accession list file (one sample/run accession per line, e.g. SRR5947837; downloaded from the SRA run selector for a BioProject such as PRJNA389280)
- Output directory path for FASTQ files

## Outputs

- Raw FASTQ files per sample (converted from .SRA format via fastq-dump), covering shotgun metagenomic and/or metatranscriptomic reads

## How to apply

Go to the SRA run selector for the BioProject of interest and download its accession list; for the Schirmer et al. (2018) IBD cohort this is BioProject PRJNA389280, whose list can be limited (e.g. to 4 CD and 4 UC samples for the tutorial). Run the `bigmap-download` entry point with `-A` pointing at the accession list file and `-O` pointing at the output directory; the module first downloads samples in .SRA format and then converts them to .fastq using fastq-dump (from the ncbi/sra-tools toolkit). The resulting FASTQ files are the direct input for `bigmap-map`, which aligns metagenomic/metatranscriptomic reads against representative GCF sequences. Installation of BiG-MAP itself is via conda (`conda env create -n bigmap -c conda-forge -c bioconda bigmap`), and the tool has only been tested on Ubuntu.

## Related tools

- **BiG-MAP (bigmap-download module)** (Entry point that takes the accession list and downloads plus converts the cohort's metagenomic/metatranscriptomic reads from SRA) — github.com/medema-group/BiG-MAP
- **ncbi/sra-tools** (Provides fastq-dump, used to convert downloaded .SRA files into .fastq)

## Examples

```
bigmap-download -A Acc_list.txt -O /usr001/fastq/schirmer/
```

## Evaluation signals

- A .fastq file is produced for every accession listed in the input file (e.g. 8 FASTQs for the 8-accession tutorial list from PRJNA389280).
- FASTQ read counts and per-sample metadata match the corresponding SRA records (CD vs. non-IBD/UC samples distinguishable).
- No intermediate .SRA files remain required — files are usable directly as `bigmap-map` input for GCF abundance/expression quantification.
- Output directory contains the files under the path passed to `-O`, organized per the downloaded samples.

## Limitations

- BiG-MAP has only been tested on Ubuntu; running bigmap-download on other platforms is unverified.
- Data must be present in the SRA database — accession lists pointing to unavailable records will fail.
- The full tutorial restricts to a subsample (8 metagenomic runs: 4 CD, 4 UC) purely to speed up the analysis; reproducing the full paper's results requires the complete PRJNA389280 accession list, including metatranscriptomic runs for expression profiling.

## Evidence

- [readme] This script is created to easily download the metagenomic and/or metatranscriptomic samples available in the SRA repository (https://www.ncbi.nlm.nih.gov/sra).: [evidence span withheld — anchor status: external_doc]
- [readme] First, the samples are downloaded in .SRA format, and then they are converted into .fastq using fastq-dump.: [evidence span withheld — anchor status: external_doc]
- [intro] BiG-MAP.download, which uses a list of SRA accession numbers to download the shotgun data if present in the SRA database: "BiG-MAP.download, which uses a list of SRA accession numbers to download the shotgun data if present in the SRA database"
