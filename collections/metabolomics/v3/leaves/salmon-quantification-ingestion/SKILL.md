---
name: salmon-quantification-ingestion
description: Use when RNA-seq transcript-level quantification files (Salmon quant.sf)
  are available and the goal is isoform-level differential expression plus isoform
  switch detection and functional annotation, as in the MOI isoform sub-workflow.
when_to_use_negative:
- When the analysis target is gene-level differential expression from a filtered/normalized/batch-corrected
  count matrix — use the edgeR/DESeq2/RankProd module instead
- When raw FASTQ files have not yet been quantified — Salmon quantification must run
  first before this ingestion step
- When the input is miRNA or other non-isoform-resolved expression data where isoform
  switch analysis is not applicable
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Salmon
  role: Produces the quant.sf quantification files that are ingested into isoformSwitchAnalyzeR
  repo: https://github.com/nf-core/modules
- name: ISOformSwitchAnalyzeR
  role: Consumes quant.sf files to perform isoform- and gene-level differential expression
    and detect genome-wide isoform switches
- name: CPAT
  role: Annotates coding potential of transcripts flagged by isoform switch analysis
- name: Pfam
  role: Annotates homology with protein domains for isoform transcripts
- name: signalP
  role: Detects presence of signaling sequences in isoform transcripts
- name: MultiOmicsIntegrator (MOI)
  role: Nextflow pipeline hosting the isoform sub-workflow; invoked with params_isoforms.yml
    and a Docker profile
  repo: https://github.com/ASAGlab/moi
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:178cf10f2310a3d28c8e2a7595015fa58a748c762a7ab5b46ab0c5c605d3e15f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/modules
  tools:
  - CPAT
  - DESeq2
  - ISOformSwitchAnalyzeR
  - MultiOmicsIntegrator (MOI)
  - Pfam
  - RankProd
  - Salmon
  - edgeR
  - signalP
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# salmon-quantification-ingestion

## Summary

Ingest Salmon quant.sf quantification files into isoformSwitchAnalyzeR within the MultiOmicsIntegrator (MOI) Nextflow pipeline, enabling differential expression at both isoform and gene levels and genome-wide isoform switch detection.

## When to use

Use when RNA-seq transcript-level quantification files (Salmon quant.sf) are available and the goal is isoform-level differential expression plus isoform switch detection and functional annotation, as in the MOI isoform sub-workflow.

## When NOT to use

- When the analysis target is gene-level differential expression from a filtered/normalized/batch-corrected count matrix — use the edgeR/DESeq2/RankProd module instead
- When raw FASTQ files have not yet been quantified — Salmon quantification must run first before this ingestion step
- When the input is miRNA or other non-isoform-resolved expression data where isoform switch analysis is not applicable

## Inputs

- Salmon quant.sf quantification files
- params_isoforms.yml configuration file for the isoform module
- Transcript sequences of interest (for CPAT/Pfam/signalP annotation)

## Outputs

- Isoform-level and gene-level differential expression results
- Genome-wide isoform switch calls
- Functional annotations of transcript coding potential (CPAT), protein domains (Pfam), and signaling sequences (signalP)
- Collected results in the pipeline output directory

## How to apply

Within the MOI pipeline, after Salmon produces quantification files, feed the quant.sf files into isoformSwitchAnalyzeR, which performs differential expression analysis on the level of both isoforms and genes and detects genome-wide isoform switches. Transcripts of interest are then functionally annotated by three companion tools: coding potential with CPAT, protein domain homology with Pfam, and signaling sequences with signalP. The isoform module runs as a separate Nextflow invocation with its own params file (params_isoforms.yml) and a Docker profile, distinct from the gene-level edgeR/DESeq2/RankProd module that operates on the preprocessed (filtered, normalized, batch-corrected) count matrix. Outputs (differential expression results and isoform switch functional annotations) are collected into the output directory for downstream inspection and use.

## Related tools

- **Salmon** (Produces the quant.sf quantification files that are ingested into isoformSwitchAnalyzeR) — https://github.com/nf-core/modules
- **ISOformSwitchAnalyzeR** (Consumes quant.sf files to perform isoform- and gene-level differential expression and detect genome-wide isoform switches)
- **CPAT** (Annotates coding potential of transcripts flagged by isoform switch analysis)
- **Pfam** (Annotates homology with protein domains for isoform transcripts)
- **signalP** (Detects presence of signaling sequences in isoform transcripts)
- **MultiOmicsIntegrator (MOI)** (Nextflow pipeline hosting the isoform sub-workflow; invoked with params_isoforms.yml and a Docker profile) — https://github.com/ASAGlab/moi

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- quant.sf files from Salmon exist as upstream inputs before isoformSwitchAnalyzeR runs
- isoformSwitchAnalyzeR reports differential expression results at both the isoform level and the gene level
- Genome-wide isoform switch detection results are produced and collected in the output directory
- Transcripts of interest receive annotations from all three tools: CPAT (coding potential), Pfam (protein domains), and signalP (signaling sequences)
- The module runs with its dedicated params_isoforms.yml and container profile rather than the gene-level DE params.yml

## Limitations

- Software versions for the isoform analysis tools are not specified in the article's methods table
- The pipeline requires Nextflow version 23.04.2.5870 specifically, and per-process containers (Docker/Singularity) for reproducibility
- Isoform analysis depends on transcript-level quantification; it cannot be run directly on gene count matrices

## Evidence

- [intro] Salmon produces quant.sf consumed by isoformSwitchAnalyzeR: "It then employs [salmon](../modules/nf-core/salmon) in order to obtain quantification files"
- [intro] isoformSwitchAnalyzeR performs isoform and gene DE: "After that [isoformSwitchAnalyzer] is used, which takes these quantification files and performs differential expression analysis on the level of both isoforms and genes."
- [intro] Functional annotation with CPAT, Pfam, signalP: [evidence span withheld — anchor status: not_found]
- [intro] Nextflow version and containers: "The pipeline is built using [Nextflow](https://www.nextflow.io) version 23.04.2.5870 (IMPORTANT), a workflow tool to run tasks across multiple compute infrastructures"
- [readme] README command invoking isoform module with params_isoforms.yml: "nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker"
- [intro] Preprocessing before DE on count matrix: "the user can perform [filtering](../modules/local/mom_filter), [normalization](../modules/local/mom_norm/) and [batch effect](../modules/local/mom_filter/) correction"
