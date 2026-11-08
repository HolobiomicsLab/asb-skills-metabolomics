---
name: transcript-abundance-quantification
description: Use when raw RNA-seq reads (FASTQ files or NCBI SRA run codes) must be
  converted into per-sample transcript quantifications and a gene-level abundance
  matrix, with QC and adapter trimming as prerequisites.
when_to_use_negative:
- Input is already Salmon quant.sf files or a gene count matrix — skip pre-processing
  and feed the quantifications into MOI's downstream filtering/normalization steps
  directly
- The analysis requires alignment-based quantification or splice-aware mapping rather
  than quasi-mapping/selective alignment (e.g. novel junction discovery)
- Data are not RNA-seq reads (e.g. proteomics, metabolomics, or lipidomics inputs
  handled by other MOI branches)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Salmon
  role: Quantifies trimmed reads against the reference transcriptome to produce per-sample
    quant.sf quantification files
  repo: https://github.com/nf-core/modules
- name: FASTQC
  role: Pre-quantification quality control of raw reads producing per-sample QC reports
  repo: https://github.com/nf-core/modules
- name: Trim Galore
  role: Automatic adapter detection and removal on raw reads before Salmon quantification
  repo: https://github.com/nf-core/modules
- name: NCBI SRA toolkit
  role: Downloads SRA run identifiers from the sample sheet and converts runs into
    FASTQ files
- name: Nextflow
  role: Workflow engine (v23.04.2.5870, DSL2, Docker/Singularity containers) orchestrating
    the pre-processing and quantification processes
  repo: https://github.com/nf-core/configs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:14f153add0ce2f9449f05b1e53288181702b23c504595950915211036d92811d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/modules
  tools:
  - FASTQC
  - NCBI SRA toolkit
  - Nextflow
  - SRA toolkit
  - Salmon
  - Trim Galore
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# transcript-abundance-quantification

## Summary

Quantify transcript/gene abundances from RNA-seq FASTQ data using Salmon, as implemented in the MultiOmicsIntegrator (MOI) Nextflow pipeline, producing per-sample quant.sf files that are aggregated into a gene-level abundance matrix.

## When to use

Use when raw RNA-seq reads (FASTQ files or NCBI SRA run codes) must be converted into per-sample transcript quantifications and a gene-level abundance matrix, with QC and adapter trimming as prerequisites.

## When NOT to use

- Input is already Salmon quant.sf files or a gene count matrix — skip pre-processing and feed the quantifications into MOI's downstream filtering/normalization steps directly
- The analysis requires alignment-based quantification or splice-aware mapping rather than quasi-mapping/selective alignment (e.g. novel junction discovery)
- Data are not RNA-seq reads (e.g. proteomics, metabolomics, or lipidomics inputs handled by other MOI branches)

## Inputs

- Sample sheet CSV with a single column of NCBI SRA run identifiers or paths to FASTQ files
- Raw RNA-seq FASTQ files (or converted from SRA runs)
- Reference transcriptome/index for Salmon
- Pre-computed Salmon quant.sf files from an external alignment (alternative input)

## Outputs

- Per-sample FASTQC quality-control reports
- Trimmed FASTQ files after adapter removal
- Per-sample Salmon quant.sf files at $OUTDIR/salmon_genes/$sampleID/quant.sf
- Aggregated gene-level abundance matrix (gene_abundance_matrix.csv)

## How to apply

Build the sample sheet CSV with either a single column of NCBI SRA run identifiers or paths to a directory of FASTQ files; SRA runs are downloaded and converted to FASTQ with the SRA toolkit. Run FASTQC on raw reads to obtain per-sample quality-control reports, then Trim Galore to automatically detect and remove adapter contamination, yielding trimmed FASTQ files. Quantify trimmed reads against the reference transcriptome with Salmon (an nf-core/modules process in MOI's Nextflow DSL2 implementation, run in Docker/Singularity containers), producing per-sample quant.sf files at $OUTDIR/salmon_genes/$sampleID/quant.sf. Aggregate transcript-level quantifications into a gene-level abundance matrix saved as gene_abundance_matrix.csv, which then feeds MOI's downstream filtering, normalization, and batch-effect-correction steps. Salmon quant.sf output from an external alignment can be supplied directly, and individual QC steps can be skipped if reads are already trimmed.

## Related tools

- **Salmon** (Quantifies trimmed reads against the reference transcriptome to produce per-sample quant.sf quantification files) — https://github.com/nf-core/modules
- **FASTQC** (Pre-quantification quality control of raw reads producing per-sample QC reports) — https://github.com/nf-core/modules
- **Trim Galore** (Automatic adapter detection and removal on raw reads before Salmon quantification) — https://github.com/nf-core/modules
- **NCBI SRA toolkit** (Downloads SRA run identifiers from the sample sheet and converts runs into FASTQ files)
- **Nextflow** (Workflow engine (v23.04.2.5870, DSL2, Docker/Singularity containers) orchestrating the pre-processing and quantification processes) — https://github.com/nf-core/configs

## Examples

```
nextflow run nf-core/mom -c params_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Verify that a quant.sf file exists at $OUTDIR/salmon_genes/$sampleID/quant.sf for every input sample
- Check that gene_abundance_matrix.csv contains rows for annotated genes with non-negative abundance values
- Confirm FASTQC reports were generated for each raw sample and Trim Galore produced trimmed FASTQ outputs (skipped only if deliberately disabled)
- Confirm the sample sheet resolved correctly: SRA codes were downloaded/converted to FASTQ, or FASTQ paths pointed to real read files

## Limitations

- MOI does not specify software versions for FastQC/TrimGalore/Salmon in the methods table, so reproducibility depends on container pinning
- Each QC step (FASTQC, Trim Galore) can be skipped via parameters, risking untrimmed adapter contamination if misconfigured
- Salmon quantifications are transcript-level and must be aggregated to gene level; downstream pipeline steps assume this aggregation is done
- Genome/transcriptome references are resolved from igenomes_base (s3://ngi-igenomes/igenomes), so non-standard organisms may need custom indices

## Evidence

- [intro] The pipeline initially downloads SRA codes and converts the runs into fastq files.: "The pipeline initially downloads SRA codes and converts the runs into fastq files."
- [intro] FASTQC QC and Trim Galore adapter removal: "It then performs quality control with [FASTQC](../modules/nf-core/fastqc) and then automatically detects and removes adapters with [Trimgalore]"
- [intro] Salmon quantification: "It then employs [salmon](../modules/nf-core/salmon) in order to obtain quantification files"
- [methods] Sample sheet description: "This file accommodates either a solitary column of SRA codes or a directory pointing to the location of fastq files"
- [readme] Docker/Singularity reproducibility: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
