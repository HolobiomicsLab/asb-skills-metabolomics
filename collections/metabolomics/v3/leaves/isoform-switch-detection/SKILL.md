---
name: isoform-switch-detection
description: Use when you have Salmon quant.sf transcript quantifications and phenotype
  groups, and need to identify isoform switches (changes in isoform usage between
  conditions) plus functional consequences of the switched isoforms.
when_to_use_negative:
- You only need gene-level differential expression from a normalized count matrix
  — use the edgeR/DESeq2/RankProd module instead of the isoformSwitchAnalyzeR sub-workflow.
- Your quantifications are gene-level counts or alignment-based (no Salmon quant.sf
  transcript-level output), since this skill operates directly on Salmon quantification
  files.
- No transcriptome annotation is available to map switched isoforms to coding sequences,
  making CPAT/Pfam/signalP annotation impossible.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ISOformSwitchAnalyzeR
  role: 'Core engine: takes Salmon quant.sf files, performs differential expression
    at isoform and gene levels, and detects genome-wide isoform switches'
  repo: github.com/nf-core/modules
- name: Salmon
  role: Upstream producer of the quant.sf quantification files consumed by isoformSwitchAnalyzeR
  repo: github.com/nf-core/modules
- name: CPAT
  role: Annotates coding potential of the switched transcript sequences
- name: Pfam
  role: Annotates homology with protein domains of the switched transcript sequences
- name: signalP
  role: Annotates presence of signaling sequences in switched transcripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:81dbeaa14830164d2e9a649be970107c3fd801fd4d84eaf325bed0079d23f9d6
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# isoform-switch-detection

## Summary

Detect genome-wide isoform switches and perform isoform- and gene-level differential expression with isoformSwitchAnalyzeR on Salmon quant.sf files, then functionally annotate switched transcripts for coding potential (CPAT), protein domains (Pfam), and signal sequences (signalP) within the MultiOmicsIntegrator (MOI) pipeline.

## When to use

Use when you have Salmon quant.sf transcript quantifications and phenotype groups, and need to identify isoform switches (changes in isoform usage between conditions) plus functional consequences of the switched isoforms.

## When NOT to use

- You only need gene-level differential expression from a normalized count matrix — use the edgeR/DESeq2/RankProd module instead of the isoformSwitchAnalyzeR sub-workflow.
- Your quantifications are gene-level counts or alignment-based (no Salmon quant.sf transcript-level output), since this skill operates directly on Salmon quantification files.
- No transcriptome annotation is available to map switched isoforms to coding sequences, making CPAT/Pfam/signalP annotation impossible.

## Inputs

- Salmon quant.sf quantification files
- Phenotype/metadata file defining sample groups
- Transcript sequences of the isoforms of interest
- params.yml for the isoformSwitchAnalyzeR module (in/out paths, parameters)

## Outputs

- Genome-wide isoform switch results and isoform/gene-level differential expression tables
- CPAT coding potential annotations
- Pfam protein domain annotations
- signalP signaling sequence annotations
- Per-module intermediate text tables and RData objects in the output directory

## How to apply

After Salmon has produced quant.sf files, run the isoformSwitchAnalyzeR module of MultiOmicsIntegrator to perform differential expression at both the isoform and gene levels and to detect genome-wide isoform switches. For the transcript sequences of interest, annotate coding potential with CPAT, homology with protein domains with Pfam, and presence of signaling sequences with signalP. Configuration of the module's paths and parameters is done through the module's params.yml file, and intermediate results are written as text tables and RData objects before final annotations are collected into the output directory. The expected biological readout is per-isoform relative expression and isoform usage fraction alongside the functional annotation of each switched isoform, as illustrated in the article's SNCA mRNA isoform example.

## Related tools

- **ISOformSwitchAnalyzeR** (Core engine: takes Salmon quant.sf files, performs differential expression at isoform and gene levels, and detects genome-wide isoform switches) — github.com/nf-core/modules
- **Salmon** (Upstream producer of the quant.sf quantification files consumed by isoformSwitchAnalyzeR) — github.com/nf-core/modules
- **CPAT** (Annotates coding potential of the switched transcript sequences)
- **Pfam** (Annotates homology with protein domains of the switched transcript sequences)
- **signalP** (Annotates presence of signaling sequences in switched transcripts)

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Salmon quant.sf files exist and are successfully ingested by the isoformSwitchAnalyzeR step (workflow step 3 of the feature-acquisition stage).
- Differential expression results are produced at BOTH the isoform and gene levels, not only gene level.
- CPAT, Pfam, and signalP annotations are present for the transcript sequences of interest.
- Per-module intermediate results are written as text tables and RData objects before collection into the output directory.
- Reported isoforms include relative expression and isoform usage fraction (as in the SNCA mRNA isoform figure).

## Limitations

- Software versions for the annotation tools are not specified in the article's methods table.
- The functional annotation sub-workflow only runs on the transcript 'sequences of interest', so genome-wide switching without prior sequence extraction will not automatically receive CPAT/Pfam/signalP annotations.
- Switch detection quality depends on the upstream Salmon quantifications produced by the MOI pipeline's FASTQC → Trim Galore → Salmon front-end.

## Evidence

- [intro] isoformSwitchAnalyzeR consumes Salmon quant files for isoform/gene DE: "After that [isoformSwitchAnalyzer] is used, which takes these quantification files and performs differential expression analysis on the level of both isoforms and genes."
- [intro] Functional annotation of switched transcripts with CPAT, Pfam, signalP: [evidence span withheld — anchor status: not_found]
- [intro] SNCA isoform annotation shows coding potential, domains, and isoform usage fraction: "Different isoforms of SNCA mRNA are detected and annotated with respect to their coding potential and protein domains. Moreover, the relative expression of the gene is displayed"
- [readme] MOI is the surrounding best-practice pipeline hosting this sub-workflow: "multiOmicsIntegrator is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
