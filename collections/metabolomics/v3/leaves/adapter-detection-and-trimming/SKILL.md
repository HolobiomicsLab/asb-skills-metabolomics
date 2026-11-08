---
name: adapter-detection-and-trimming
description: Use when the input is raw FASTQ files (or an SRA run-accession samplesheet
  to be converted to FASTQ) and adapter contamination must be detected and removed
  automatically before quantification with Salmon.
when_to_use_negative:
- The input FASTQ files are already adapter-trimmed, or pre-computed Salmon quant.sf
  files from an external alignment are being supplied directly (MOI allows bypassing
  these QC steps)
- The task belongs to the pipeline's proteomics, lipidomics, or multi-omics integration
  branches, which have no FASTQ adapter-trimming stage
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3293
tools:
- name: Trim Galore
  role: Automatically detects and removes adapter contamination from raw reads, producing
    the trimmed FASTQ files passed to Salmon; installed as an nf-core/modules process
  repo: https://github.com/nf-core/modules
- name: FASTQC
  role: Runs immediately before trimming to generate per-sample quality-control reports
    that inform whether trimming is needed; also installed from nf-core/modules
  repo: https://github.com/nf-core/modules
- name: SRA toolkit
  role: Downloads and converts SRA run codes from the samplesheet into FASTQ files
    when raw FASTQ is not provided locally
- name: Nextflow
  role: Workflow engine (version 23.04.2.5870) that orchestrates the FastQC and Trim
    Galore processes with Docker/Singularity containerisation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:8a597a478f2e73131c48eceda00231f438c0b4207c1192534e44985909192de3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/modules
  tools:
  - FASTQC
  - Nextflow
  - SRA toolkit
  - Trim Galore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# adapter-detection-and-trimming

## Summary

Automatic adapter detection and removal from raw RNA-seq FASTQ reads using Trim Galore, preceded by FastQC quality control, as implemented in the MOI Nextflow pipeline's RNA-seq pre-processing stage before Salmon quantification.

## When to use

Use when the input is raw FASTQ files (or an SRA run-accession samplesheet to be converted to FASTQ) and adapter contamination must be detected and removed automatically before quantification with Salmon.

## When NOT to use

- The input FASTQ files are already adapter-trimmed, or pre-computed Salmon quant.sf files from an external alignment are being supplied directly (MOI allows bypassing these QC steps)
- The task belongs to the pipeline's proteomics, lipidomics, or multi-omics integration branches, which have no FASTQ adapter-trimming stage

## Inputs

- Raw paired/single-end FASTQ files from a directory specified in the MOI samplesheet CSV
- NCBI SRA run accession codes supplied as a single-column samplesheet CSV (converted to FASTQ via SRA toolkit)

## Outputs

- Trimmed FASTQ files with detected adapters removed
- Per-sample FastQC quality-control reports on the raw reads
- Downstream Salmon quant.sf quantification files per sample (e.g. $OUTDIR/salmon_genes/$sampleID/quant.sf)

## How to apply

First obtain FASTQ input: read the MOI samplesheet CSV, which holds either a single column of SRA run codes or a path to a directory of FASTQ files, and convert SRA runs to FASTQ with the SRA toolkit. Run FastQC on the raw reads to generate per-sample quality-control reports so adapter content and quality profiles are visible before trimming. Then run Trim Galore, which automatically detects adapters rather than requiring a user-specified adapter sequence, to yield trimmed FASTQ files. Feed the trimmed FASTQ into Salmon quantification, which writes per-sample quant.sf files (e.g. $OUTDIR/salmon_genes/$sampleID/quant.sf) that are aggregated into a gene-level abundance matrix. In MOI each pre-processing QC step (FastQC, Trim Galore) can be individually skipped via configuration, and externally produced Salmon output can be supplied directly, so decide the trimming step's inclusion from the state of the input reads. The pipeline executes these tools as nf-core/modules processes inside Docker/Singularity containers under Nextflow for reproducibility.

## Related tools

- **Trim Galore** (Automatically detects and removes adapter contamination from raw reads, producing the trimmed FASTQ files passed to Salmon; installed as an nf-core/modules process) — https://github.com/nf-core/modules
- **FASTQC** (Runs immediately before trimming to generate per-sample quality-control reports that inform whether trimming is needed; also installed from nf-core/modules) — https://github.com/nf-core/modules
- **SRA toolkit** (Downloads and converts SRA run codes from the samplesheet into FASTQ files when raw FASTQ is not provided locally)
- **Nextflow** (Workflow engine (version 23.04.2.5870) that orchestrates the FastQC and Trim Galore processes with Docker/Singularity containerisation)

## Examples

```
nextflow run nf-core/mom -c nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Per-sample FastQC QC reports exist for every raw-read input sample listed in the samplesheet
- A trimmed FASTQ output file exists for every input FASTQ sample (one-to-one correspondence)
- Adapter sequences were removed by automatic detection, not manual specification — confirm no user-supplied adapter string parameter was required
- Trimming was skipped only when explicitly configured to skip, or when Salmon quant.sf from an external alignment was provided directly
- Every sample reaching Salmon yields a quant.sf file, confirming the trimmed FASTQ were quantifiable

## Limitations

- Software versions for FastQC and Trim Galore are not specified in the article's methods table for the QC step
- Each QC step (FastQC, Trim Galore) can be skipped individually, so a run may silently produce untrimmed reads if trimming is disabled
- Trimming applies only to the RNA-seq pre-processing branch; it does not cover the pipeline's miRNA, isoform annotation, proteomics, or lipidomics handling

## Evidence

- [intro] FastQC runs before trimming and Trim Galore follows automatically: "It then performs quality control with [FASTQC](../modules/nf-core/fastqc) and then automatically detects and removes adapters with [Trimgalore]"
- [intro] Trim Galore is installed as an nf-core/modules process in the pipeline: "then automatically detects and removes adapters with [Trimgalore](../modules/nf-core/trimgalore)."
- [intro] FASTQ input may originate from SRA run codes converted by the pipeline: "The pipeline initially downloads SRA codes and converts the runs into fastq files."
- [readme] Containers make the trimming/QC steps reproducible across infrastructures: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
