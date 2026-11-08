---
name: breast-cancer-subtype-comparison-analysis
description: Use when a multi-omics breast cancer dataset (genes/proteins/lipids-metabolomics
  matrices plus a TNBC-vs-other-subtype phenotype assignment) must be analyzed per-omics
  and integrated to identify subtype-discriminating features and validated TNBC targets.
when_to_use_negative:
- Input is already an integrated or single-layer result set (e.g. only a precomputed
  DE table) rather than the raw sample sheet/count matrices MOI expects
- The phenotype of interest is not a multi-group subtype contrast representable in
  the MOI sample metadata, or the data cover omics types MOI's segments (genes, miRNA,
  isoforms, proteins, lipids) do not support
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Unified Nextflow pipeline performing per-omics analyses (genes, miRNA, isoforms,
    proteomics, lipidomics) and MCIA integration for the TNBC comparison
  repo: https://github.com/ASAGlab/moi
- name: Nextflow
  role: Workflow engine (version 23.04.2.5870) executing the MOI pipeline tasks
  repo: https://github.com/nf-core
- name: FASTQC
  role: Quality control of downloaded fastq files in the transcriptomics segment
  repo: https://github.com/nf-core/modules
- name: Trim Galore
  role: Automatic adapter detection and removal after FASTQC
  repo: https://github.com/nf-core/modules
- name: Salmon
  role: Produces quant.sf quantification files feeding isoform and gene-level analysis
  repo: https://github.com/nf-core/modules
- name: ISOformSwitchAnalyzeR
  role: Differential expression on isoform and gene level from Salmon quantifications
- name: Docker
  role: Container profile making the pipeline installation trivial and results reproducible
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:3880f03c6afd4cf6150ea8b49e5eb2f86fab391b5b23445a6176a7f81b41535a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - Docker
  - FASTQC
  - ISOformSwitchAnalyzeR
  - MultiOmicsIntegrator (MOI)
  - Nextflow
  - Salmon
  - Singularity
  - Trim Galore
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# breast-cancer-subtype-comparison-analysis

## Summary

Applies the MultiOmicsIntegrator (MOI) Nextflow pipeline to compare triple negative breast cancer (TNBC) against other BC subtypes across transcriptomics, proteomics and lipidomics layers, with MCIA integration to recover validated TNBC targets such as XBP1 and DGAT2 and phenotype-discriminating pathways.

## When to use

Use when a multi-omics breast cancer dataset (genes/proteins/lipids-metabolomics matrices plus a TNBC-vs-other-subtype phenotype assignment) must be analyzed per-omics and integrated to identify subtype-discriminating features and validated TNBC targets.

## When NOT to use

- Input is already an integrated or single-layer result set (e.g. only a precomputed DE table) rather than the raw sample sheet/count matrices MOI expects
- The phenotype of interest is not a multi-group subtype contrast representable in the MOI sample metadata, or the data cover omics types MOI's segments (genes, miRNA, isoforms, proteins, lipids) do not support

## Inputs

- Single CSV sample sheet with SRA run codes (or fastq directory) plus sample metadata defining TNBC vs other-subtype phenotype
- Directory of feature count matrices (genes, proteins, lipids/metabolomics) with a separate phenotype file for matrix-starting modules
- params.yml configuration files, one per analysis segment (genes, proteins, lipids/metabolomics, MCIA integration)

## Outputs

- Per-omics differential expression and pathway enrichment results (text tables, RData objects, plots) under the user-defined output tree, e.g. /user_defined_output_directory/genes/biotranslator/
- MCIA integration outputs: PCA of the sample space differentiating samples by phenotype with variables projected to show their contribution
- Prioritized TNBC features (e.g. XBP1, DGAT2) and enriched pathways for adhesion, proliferation and secretion

## How to apply

Prepare the single sample sheet CSV accepted by MOI containing SRA run codes (or a fastq directory) plus sample metadata defining the TNBC-vs-other-subtypes phenotype; alternatively supply the feature matrix directory with a separate phenotype file for modules starting from count matrices. Write one params.yml per analysis segment (genes, proteins, lipids/metabolomics, MCIA integration) specifying input/output paths and fine-tuning choices such as the filter (filterByExp), normalization (quantile), and batch correction method (com/sva/comsva/svacom/none). Launch the MOI Nextflow pipeline built on the nf-core template with a Docker or Singularity profile: SRA runs are downloaded to fastq, QC'd with FASTQC, adapter-trimmed with Trim Galore, and quantified with Salmon; count matrices then undergo filtering, quantile normalization, and ComBat/SVA batch-effect correction per params settings. Run differential expression with edgeR, DESeq2 or RankProd as configured, then pathway enrichment with clusterProfiler or BioTranslator, and finally the MCIA integration module across omics layers, inspecting the hierarchically organized outputs (e.g. under /user_defined_output_directory/genes/biotranslator/). Validate by confirming the output tree is populated with text tables, RData objects and plots per module and that prioritized features such as XBP1 and DGAT2 appear among differential/integrated results.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Unified Nextflow pipeline performing per-omics analyses (genes, miRNA, isoforms, proteomics, lipidomics) and MCIA integration for the TNBC comparison) — https://github.com/ASAGlab/moi
- **Nextflow** (Workflow engine (version 23.04.2.5870) executing the MOI pipeline tasks) — https://github.com/nf-core
- **FASTQC** (Quality control of downloaded fastq files in the transcriptomics segment) — https://github.com/nf-core/modules
- **Trim Galore** (Automatic adapter detection and removal after FASTQC) — https://github.com/nf-core/modules
- **Salmon** (Produces quant.sf quantification files feeding isoform and gene-level analysis) — https://github.com/nf-core/modules
- **ISOformSwitchAnalyzeR** (Differential expression on isoform and gene level from Salmon quantifications)
- **Docker** (Container profile making the pipeline installation trivial and results reproducible)

## Examples

```
nextflow run nf-core/mom -c params_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- The output tree under the user-defined output directory is populated with text tables, RData objects and plots for each module (genes, proteins, lipids, MCIA)
- Validated TNBC targets XBP1 and DGAT2 appear among the differential/integrated results, consistent with the published Results
- MCIA reports a PCA of the sample space in which samples differentiate according to the TNBC vs other-subtype phenotype
- Enriched pathways for adhesion, proliferation and secretion are recovered, and prioritized features show potential prognostic value

## Limitations

- Software versions of the individual QC/quantification tools are not specified in the methods table
- Preprocessing choices (filterByExp, quantile normalization, ComBat/SVA) must be explicitly tuned in params.yml per segment; misconfiguration changes downstream DE and enrichment results
- Pipeline requires Nextflow version 23.04.2.5870 (flagged IMPORTANT), and depends on nf-core/modules and nf-core/configs for reproducible execution

## Evidence

- [intro] SRA download and conversion of runs into fastq files: "The pipeline initially downloads SRA codes and converts the runs into fastq files."
- [intro] Preprocessing options: "mom_batch_method_genes          = "com" # com for combat, sva,  comsva for combat & sva, svacom for sva and comba, none"
- [intro] Differential expression algorithms: "Now,is time to perform differential expression analysis. We provide three different algorithms for that, which we describe below."
- [intro] Pathway enrichment step: "Last step of the analysis is to perform pathway enrichment analysis with [clusterprofiler](../modules/local/clusterprofiler) or biotranslator"
- [intro] MCIA integration result: "MCIA reports the PCA of the sample space where we can see how samples differentiate according to the phenotype of interest."
- [readme] Pipeline reproducibility: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
- [methods] Sample sheet specification: "This file accommodates either a solitary column of SRA codes or a directory pointing to the location of fastq files"
