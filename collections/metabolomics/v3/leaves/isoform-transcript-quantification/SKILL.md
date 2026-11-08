---
name: isoform-transcript-quantification
description: Use when RNA-seq FASTQ files (or SRA runs) must be quantified at the
  isoform level — e.g. to detect differentially expressed isoforms or isoform switching
  — rather than only gene-level counts.
when_to_use_negative:
- Gene-level differential expression only is needed — use the MOI gene sub-workflow
  (edgeR/DESeq2/RankProduct) instead of Salmon+isoformSwitchAnalyzeR
- Input is already a quant.sf count matrix from another quantifier rather than raw
  FASTQ/SRA runs
- Data are miRNA-seq, proteomics, or metabolomics — the isoform module is specific
  to mRNA isoform quantification
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3308
tools:
- name: Salmon
  role: Produces quant.sf isoform-level quantification files from adapter-trimmed
    FASTQ reads
- name: Trim Galore
  role: Automatically detects and removes adapters before Salmon quantification
- name: FASTQC
  role: Quality control of raw FASTQ reads prior to trimming and quantification
- name: ISOformSwitchAnalyzeR
  role: Consumes Salmon quant.sf files to perform differential isoform/gene expression
    and isoform-switching analysis
- name: MultiOmicsIntegrator (MOI)
  role: Nextflow pipeline wrapper orchestrating SRA download, QC, trimming, and Salmon
    quantification
  repo: https://github.com/nf-core/modules
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:84597a169effaec187f9481c29e2ae7fbd893e744b9a6531d793b4de87cf3935
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/modules
  tools:
  - CPAT
  - Docker
  - FASTQC
  - ISOformSwitchAnalyzeR
  - MultiOmicsIntegrator (MOI)
  - Pfam
  - SRA toolkit
  - Salmon
  - Singularity
  - Trim Galore
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# Isoform transcript quantification

## Summary

Quantifies transcript-level (isoform) abundances from RNA-seq FASTQ data using Salmon within the MOI Nextflow pipeline, producing quant.sf files that feed downstream isoformSwitchAnalyzeR differential isoform and isoform-switching analysis.

## When to use

Use when RNA-seq FASTQ files (or SRA runs) must be quantified at the isoform level — e.g. to detect differentially expressed isoforms or isoform switching — rather than only gene-level counts.

## When NOT to use

- Gene-level differential expression only is needed — use the MOI gene sub-workflow (edgeR/DESeq2/RankProduct) instead of Salmon+isoformSwitchAnalyzeR
- Input is already a quant.sf count matrix from another quantifier rather than raw FASTQ/SRA runs
- Data are miRNA-seq, proteomics, or metabolomics — the isoform module is specific to mRNA isoform quantification

## Inputs

- RNA-seq FASTQ files (or SRA run identifiers in a metadata CSV, downloaded via the SRA toolkit)
- QC-checked reads after FASTQC and Trim Galore adapter removal
- params_isoforms.yml specifying input/output directories

## Outputs

- Salmon quant.sf quantification files per sample
- isoformSwitchAnalyzeR differential isoform and gene expression results
- genome-wide isoform-switching analysis results (plots, text, RData) under the isoforms/ output directory

## How to apply

Run the MOI isoform sub-workflow with `nextflow run nf-core/mom -c params_isoforms.yml -profile docker`. The pipeline first downloads SRA runs and converts them to FASTQ, performs QC with FASTQC, then automatically detects and removes adapters with Trim Galore. It then employs Salmon to produce quant.sf quantification files per sample; these are the required input format for isoformSwitchAnalyzeR, which performs differential expression at both isoform and gene level. Containers (Docker/Singularity) are used to guarantee reproducibility; provide a metadata CSV with a single column of SRA run identifiers or a directory of FASTQ files plus phenotype metadata. Accept the run when quant.sf files and the organized isoform outputs appear under the user-defined output directory's isoforms/ subpath.

## Related tools

- **Salmon** (Produces quant.sf isoform-level quantification files from adapter-trimmed FASTQ reads)
- **Trim Galore** (Automatically detects and removes adapters before Salmon quantification)
- **FASTQC** (Quality control of raw FASTQ reads prior to trimming and quantification)
- **ISOformSwitchAnalyzeR** (Consumes Salmon quant.sf files to perform differential isoform/gene expression and isoform-switching analysis)
- **MultiOmicsIntegrator (MOI)** (Nextflow pipeline wrapper orchestrating SRA download, QC, trimming, and Salmon quantification) — https://github.com/nf-core/modules

## Examples

```
nextflow run nf-core/mom -c params_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- quant.sf files are produced for every input sample and organized hierarchically under /user_defined_output_directory/isoforms/
- Differential isoform calls for known biology reproduce, e.g. SNCA isoforms in Parkinson's disease RNA-seq data
- informative plots plus text and RData intermediate results for each step are generated
- reads pass FASTQC QC and adapter content is removed by Trim Galore before quantification
- run completes under Docker/Singularity containers from nf-core/modules with no process failures

## Limitations

- Exact software versions are not specified in the article's module table, so version pinning must be handled via containers/configs
- The isoform workflow depends on Salmon's quasi-mapping; transcriptome reference choices are delegated to the igenomes_base parameter
- CPAT, Pfam, and signalP annotations of transcripts are separate post-quantification steps, not part of the quantification itself
- Release metadata (date, changelog sections) in the source repo README are placeholders, complicating reproducibility bookkeeping

## Evidence

- [intro] Salmon produces the quant.sf files consumed by isoformSwitchAnalyzeR: "It then employs [salmon](../modules/nf-core/salmon) in order to obtain quantification files"
- [intro] Pre-quantification QC and adapter trimming steps in the pipeline: "It then performs quality control with [FASTQC](../modules/nf-core/fastqc) and then automatically detects and removes adapters with [Trimgalore]"
- [intro] isoformSwitchAnalyzeR consumes the quantification files for differential isoform and gene analysis: "After that [isoformSwitchAnalyzer] is used, which takes these quantification files and performs differential expression analysis on the level of both isoforms and genes"
- [intro] SRA runs are the entry point converted to FASTQ before quantification: "The pipeline initially downloads SRA codes and converts the runs into fastq files."
- [readme] Containerization requirement for reproducible execution: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
