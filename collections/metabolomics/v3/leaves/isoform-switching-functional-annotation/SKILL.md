---
name: isoform-switching-functional-annotation
description: Use when RNA-seq quantification is available at isoform level (Salmon
  quant.sf files) and the goal is to identify genes with differentially used isoforms
  and interpret the functional consequences of isoform switching events.
when_to_use_negative:
- Input is already gene-level count data without isoform-level quantification — Salmon
  quant.sf files from transcript-level quantification are the expected input for isoformSwitchAnalyzeR.
- The goal is simple gene-level differential expression without interest in isoform
  usage or switching events — the gene-only DE modules (edgeR, DESeq2, RankProduct)
  suffice.
edam_operation: http://edamontology.org/operation_2490
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0085
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Pipeline container that orchestrates the isoform sub-workflow, from SRA download
    to functional annotation of isoform-switching events
  repo: https://github.com/ASAGlab/moi
- name: Salmon
  role: Produces the quant.sf quantification files that feed the isoform differential
    expression and switching analysis
  repo: https://github.com/COMBINE-lab/salmon
- name: ISOformSwitchAnalyzeR
  role: Takes quant.sf files and performs differential expression analysis at the
    level of both isoforms and genes, plus genome-wide isoform-switching analysis
- name: CPAT
  role: Annotates transcript sequences regarding their coding potential
- name: Pfam
  role: Annotates transcript homology with protein domains
- name: signalP
  role: Detects the existence of any signaling sequence in the isoform sequences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:0e8ee7fbe3616a87d2b3d1e57bacbed520ca128db90bc6f34dd2a21643b44808
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
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
  - signalP
  edam_operation: http://edamontology.org/operation_2490
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# isoform-switching-functional-annotation

## Summary

Detection of differentially expressed isoforms and genome-wide isoform-switching events from Salmon quantification files, followed by functional annotation of the affected transcript sequences. Implemented in the MOI Nextflow pipeline via ISOformSwitchAnalyzeR plus CPAT, Pfam, and signalP.

## When to use

Use when RNA-seq quantification is available at isoform level (Salmon quant.sf files) and the goal is to identify genes with differentially used isoforms and interpret the functional consequences of isoform switching events.

## When NOT to use

- Input is already gene-level count data without isoform-level quantification — Salmon quant.sf files from transcript-level quantification are the expected input for isoformSwitchAnalyzeR.
- The goal is simple gene-level differential expression without interest in isoform usage or switching events — the gene-only DE modules (edgeR, DESeq2, RankProduct) suffice.

## Inputs

- Metadata CSV with a single column of SRA run identifiers (or a directory of fastq files) plus phenotype metadata
- params_isoforms.yml parameter file specifying input/output directories
- Salmon quant.sf isoform quantification files

## Outputs

- Differential isoform and gene expression results from isoformSwitchAnalyzeR
- Genome-wide isoform-switching analysis results (alternative transcript start/end events, UTR length changes)
- Functional annotations of transcript sequences: coding potential (CPAT), protein domain homology (Pfam), signaling sequences (signalP)
- Informative plots plus text and RData intermediate results for each step

## How to apply

Take the quant.sf quantification files produced by Salmon and pass them to ISOformSwitchAnalyzeR within the MOI pipeline, which performs differential expression analysis at the level of both isoforms and genes and a genome-wide isoform-switching analysis, revealing events such as shifts in alternative transcript start/end sites and changes in UTR lengths (as observed in the Dumitriu Parkinson's disease dataset, where isoform switching was detected for SNCA, GSDMD, OPA1, RIC1, and TERF1). For each significant isoform, annotate the transcript sequence regarding its coding potential with CPAT, its homology with protein domains with Pfam, and the existence of any signaling sequence with signalP. Input is configured via a metadata CSV containing a single column of SRA run identifiers (or a directory of fastq files) plus phenotype metadata, and a params_isoforms.yml specifying input/output directories; launch the pipeline with Nextflow using Docker or Singularity containers so that each process runs in its own container. Outputs are organized hierarchically under a user-defined output directory (e.g. /user_defined_output_directory/isoforms/), matching the MOI output convention.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Pipeline container that orchestrates the isoform sub-workflow, from SRA download to functional annotation of isoform-switching events) — https://github.com/ASAGlab/moi
- **Salmon** (Produces the quant.sf quantification files that feed the isoform differential expression and switching analysis) — https://github.com/COMBINE-lab/salmon
- **ISOformSwitchAnalyzeR** (Takes quant.sf files and performs differential expression analysis at the level of both isoforms and genes, plus genome-wide isoform-switching analysis)
- **CPAT** (Annotates transcript sequences regarding their coding potential)
- **Pfam** (Annotates transcript homology with protein domains)
- **signalP** (Detects the existence of any signaling sequence in the isoform sequences)

## Examples

```
nextflow run nf-core/mom -c params_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- All Salmon quant.sf files are produced downstream of FASTQC QC and Trim Galore adapter trimming
- isoformSwitchAnalyzeR yields differential expression calls at both the isoform and gene level from the quant.sf inputs
- Genome-wide isoform-switching output reports alternative transcript start/end events and UTR length changes
- Significant isoforms receive CPAT coding-potential, Pfam domain, and signalP signal-sequence annotations
- Outputs are organized hierarchically under /user_defined_output_directory/isoforms/ matching the MOI output convention

## Limitations

- Software versions for FastQC, trimgalore, Salmon, isoformSwitchAnalyzeR, CPAT, Pfam, and signalP are not specified in the paper.
- Pipeline requires Docker or Singularity availability; the Nextflow version is pinned at 23.04.2.5870 (IMPORTANT).
- The release/dependency sections of the repository changelog (Added/Fixed/Dependencies/Deprecated) were left empty, so operational caveats beyond those documented are unknown.

## Evidence

- [intro] It then employs [salmon](../modules/nf-core/salmon) in order to obtain quantification files: "It then employs [salmon](../modules/nf-core/salmon) in order to obtain quantification files"
- [intro] After that [isoformSwitchAnalyzer] is used, which takes these quantification files and performs differential expression analysis on the level of both isoforms and genes.: "After that [isoformSwitchAnalyzer] is used, which takes these quantification files and performs differential expression analysis on the level of both isoforms and genes."
- [intro] their sequences are annotated regarding their coding potential [CPAT](../modules/local/cpat), their homology with protein domains [Pfam], and the existense of any signaling sequence [signalP]: [evidence span withheld — anchor status: not_found]
- [readme] It uses Docker/Singularity containers making installation trivial and results highly reproducible.: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
