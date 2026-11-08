---
name: rna-seq-differential-expression-analysis
description: Use when raw RNA-seq data (SRA run accessions or FASTQ files) or a gene/miRNA/isoform
  count matrix must be turned into a ranked list of differentially expressed features
  between phenotypes (e.g. TNBC vs other BC subtypes) as part of a multi-omics workflow.
when_to_use_negative:
- Input is already a processed feature/protein/lipid matrix with no RNA-seq counts
  — use the proteomics, lipidomics/metabolomics, or MCIA integration modules instead
- The goal is isoform-level switching and functional annotation of transcripts (coding
  potential, Pfam domains, signalP) rather than gene-level differential expression
  — that is the dedicated isoformSwitchAnalyzeR module
- Only one phenotype/sample group exists with no comparison of interest, leaving no
  DE contrast to test
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Nextflow-based pipeline wrapping the full RNA-seq DE workflow from sample
    sheet to DE tables
  repo: https://github.com/ASAGlab/MOI--An-integrated-solution-for-omics-analyses
- name: Salmon
  role: Transcript quantification producing quant.sf files that feed the DE analysis
- name: FASTQC
  role: Per-sample quality control of FASTQ input prior to trimming and quantification
  repo: https://github.com/nf-core/modules
- name: Trim Galore
  role: Automatic adapter detection and removal on FASTQ files
- name: ISOformSwitchAnalyzeR
  role: Differential expression at isoform and gene level from Salmon quantification
    files
- name: Nextflow
  role: Workflow engine (version 23.04.2.5870) executing the DE modules with Docker/Singularity
    containers
  repo: https://github.com/nf-core/configs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:2383d06842423c03f7d55ae9f8fc308bf901b3684eaaacd84c7ed955f79a9456
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/MOI--An-integrated-solution-for-omics-analyses
  tools:
  - Docker
  - FASTQC
  - ISOformSwitchAnalyzeR
  - MultiOmicsIntegrator (MOI)
  - Nextflow
  - Salmon
  - Singularity
  - Trim Galore
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# rna-seq-differential-expression-analysis

## Summary

Differential expression analysis of RNA-seq data as implemented in the MultiOmicsIntegrator (MOI) Nextflow pipeline: quantification with Salmon, count-matrix preprocessing (filterByExp filtering, quantile normalization, ComBat/SVA batch correction), and DE testing with edgeR, DESeq2, or RankProduct.

## When to use

Use when raw RNA-seq data (SRA run accessions or FASTQ files) or a gene/miRNA/isoform count matrix must be turned into a ranked list of differentially expressed features between phenotypes (e.g. TNBC vs other BC subtypes) as part of a multi-omics workflow.

## When NOT to use

- Input is already a processed feature/protein/lipid matrix with no RNA-seq counts — use the proteomics, lipidomics/metabolomics, or MCIA integration modules instead
- The goal is isoform-level switching and functional annotation of transcripts (coding potential, Pfam domains, signalP) rather than gene-level differential expression — that is the dedicated isoformSwitchAnalyzeR module
- Only one phenotype/sample group exists with no comparison of interest, leaving no DE contrast to test

## Inputs

- csv sample sheet with SRA run identifiers or a directory of FASTQ files plus sample metadata defining the phenotype
- gene/miRNA/isoform count matrix directory plus a phenotype file (for modules starting from count matrices)
- params.yml configuration file specifying input/output paths and algorithm choices

## Outputs

- differential expression result tables per algorithm (edgeR, DESeq2, or RankProduct)
- Salmon quant.sf quantification files
- text tables, RData objects, and plots per module under the user-defined output directory
- prioritized DE features feeding pathway enrichment (clusterProfiler/BioTranslator) and MCIA integration

## How to apply

Start from either a single csv sample sheet (SRA codes or fastq directory plus phenotype metadata such as TNBC-vs-other-subtypes) or a directory of feature matrices plus a phenotype file. The pipeline downloads SRA runs to FASTQ, runs FastQC quality control, removes adapters with Trim Galore, and quantifies with Salmon to obtain quant.sf files. Preprocess each count matrix by expression-based filtering with filterByExp (mom_filt_method_genes = 'filterByExp' or a user cutoff), quantile normalization (mom_norm_method_genes = 'quantile'), and batch effect correction configured via mom_batch_method_genes ('com' for ComBat, 'sva', 'comsva', 'svacom', or 'none'). Then run differential expression with one of three configured algorithms — edgeR, DESeq2, or RankProduct — choosing per the study design in params.yml. Downstream, feed the DE results to pathway enrichment (clusterProfiler or BioTranslator) or the MCIA integration module; in the CLLE breast cancer application this recovered validated TNBC targets XBP1 and DGAT2.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Nextflow-based pipeline wrapping the full RNA-seq DE workflow from sample sheet to DE tables) — https://github.com/ASAGlab/MOI--An-integrated-solution-for-omics-analyses
- **Salmon** (Transcript quantification producing quant.sf files that feed the DE analysis)
- **FASTQC** (Per-sample quality control of FASTQ input prior to trimming and quantification) — https://github.com/nf-core/modules
- **Trim Galore** (Automatic adapter detection and removal on FASTQ files)
- **ISOformSwitchAnalyzeR** (Differential expression at isoform and gene level from Salmon quantification files)
- **Nextflow** (Workflow engine (version 23.04.2.5870) executing the DE modules with Docker/Singularity containers) — https://github.com/nf-core/configs

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Output tree under the user-defined output directory is populated with text tables, RData objects, and plots for the DE module
- Pipeline completes with Nextflow 23.04.2.5870 under the docker/singularity profile without process failures
- Count matrices were filtered by filterByExp or the configured cutoff and normalized with the quantile method before DE, per params.yml settings
- Known biology is recovered — in the CLLE TNBC case, validated targets XBP1 and DGAT2 appear among the differential/integrated results
- Salmon quant.sf files exist for every sample and isoformSwitchAnalyzeR runs on them when isoform-level analysis is enabled

## Limitations

- Pipeline requires Nextflow specifically at version 23.04.2.5870 (marked IMPORTANT in the README); other versions may not be supported
- DE algorithm choice (edgeR, DESeq2, RankProduct) is user-configured and results depend on that selection
- Batch correction choice (com/sva/comsva/svacom/none) must match the batch structure; misconfiguration can distort DE results
- Article's software-version table for QC tools lacks explicit versions (flagged as a missing signal)

## Evidence

- [intro] It then employs salmon in order to obtain quantification files: "It then employs [salmon](../modules/nf-core/salmon) in order to obtain quantification files"
- [intro] Three DE algorithms provided: "Now,is time to perform differential expression analysis. We provide three different algorithms for that, which we describe below."
- [intro] Count matrix preprocessing parameters: "mom_filt_method_genes           = "filterByExp"  # filterByExp or choose a cutoff value"
- [intro] Batch correction options: "mom_batch_method_genes          = "com" # com for combat, sva,  comsva for combat & sva, svacom for sva and comba, none"
- [readme] Reproducibility via containers: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
