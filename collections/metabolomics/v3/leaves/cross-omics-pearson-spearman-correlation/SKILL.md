---
name: cross-omics-pearson-spearman-correlation
description: Use when multiple aligned omics feature matrices (e.g. mRNA, miRNA, isoform,
  proteomics, lipidomics tables) are available after MOI's MCIA integration step and
  noisy feature-to-feature connections must be filtered out before pathway enrichment
  analysis.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/SRA runs) — correlation comes after QC (FASTQC,
  Trim Galore), Salmon quantification, and count-matrix filtering/normalization/batch
  correction, not before
- The goal is the primary data-driven integration of samples across omics — MOI uses
  MCIA for that, reporting the sample-space PCA and pseudo-eigen values; correlation
  is a downstream connection-filtering step
- Only a single omics layer is available — pairwise cross-omics correlation requires
  at least two aligned feature matrices
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: R package stats
  role: Computes the pairwise Pearson or Spearman correlations between the omics feature
    matrices
- name: MultiOmicsIntegrator (MOI)
  role: Nextflow pipeline that loads the aligned feature matrices, orchestrates the
    correlation-based filtering step, and writes its text/RData outputs into the user-defined
    output directory
  repo: ASAGlab/moi
- name: clusterProfiler
  role: Consumes the correlation-filtered features for pathway enrichment performed
    after the integration step of MCIA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:62b294c78e3c280388e45e64be03e0b96f37811f5c945dfc7793f3abec8e33fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - BioTranslator
  - MultiOmicsIntegrator (MOI)
  - R package stats
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-omics-pearson-spearman-correlation

## Summary

Compute pairwise Pearson or Spearman correlations between per-omics feature matrices (genes, miRNAs, isoforms, proteins, lipids) within the MultiOmicsIntegrator (MOI) pipeline to filter illuminating connections and mitigate noise before downstream pathway enrichment with clusterProfiler or BioTranslator.

## When to use

Use when multiple aligned omics feature matrices (e.g. mRNA, miRNA, isoform, proteomics, lipidomics tables) are available after MOI's MCIA integration step and noisy feature-to-feature connections must be filtered out before pathway enrichment analysis.

## When NOT to use

- Input is raw sequencing data (FASTQ/SRA runs) — correlation comes after QC (FASTQC, Trim Galore), Salmon quantification, and count-matrix filtering/normalization/batch correction, not before
- The goal is the primary data-driven integration of samples across omics — MOI uses MCIA for that, reporting the sample-space PCA and pseudo-eigen values; correlation is a downstream connection-filtering step
- Only a single omics layer is available — pairwise cross-omics correlation requires at least two aligned feature matrices

## Inputs

- Per-omics feature matrices (genes, miRNAs, isoforms, proteins, lipids) from MOI upstream modules
- Phenotype file for the samples of interest
- params.yml with input directories governing the MOI integration module

## Outputs

- Pairwise cross-omics correlation results (Pearson or Spearman) used for connection filtering
- Filtered correlation patterns passed to clusterProfiler/BioTranslator pathway enrichment
- Intermediate results as text and RData objects
- Informative plots written under /user_defined_output_directory/<omics>/<tool>/

## How to apply

Load the per-omics feature matrices produced by MOI's upstream modules (RNA-seq counts for genes, miRNAs and isoforms including lncRNAs, plus proteomics and lipidomics tables) together with the phenotype file, using the input directories specified in the params.yml governing the integration module. Compute pairwise Pearson or Spearman correlations between the omics feature matrices with the R stats package, after the data-driven MCIA integration has projected all omics onto a common sample space. Use the correlation results to filter 'illuminating' connections and patterns, aiding noise mitigation so that downstream analyses like pathway enrichment analysis (clusterProfiler or BioTranslator) operate on confident cross-omics relationships rather than raw feature lists. Write the intermediate correlation results as text and RData objects plus informative plots into the hierarchical output structure (e.g. /user_defined_output_directory/<omics>/<tool>/). The choice of Pearson vs Spearman and any numeric correlation cutoff is left to the user; the article does not prescribe a threshold, so the decision point is between linear (Pearson) and rank-based (Spearman) association given the distribution and size of each omics matrix.

## Related tools

- **R package stats** (Computes the pairwise Pearson or Spearman correlations between the omics feature matrices)
- **MultiOmicsIntegrator (MOI)** (Nextflow pipeline that loads the aligned feature matrices, orchestrates the correlation-based filtering step, and writes its text/RData outputs into the user-defined output directory) — ASAGlab/moi
- **clusterProfiler** (Consumes the correlation-filtered features for pathway enrichment performed after the integration step of MCIA)

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Correlation text and RData objects exist under the user-defined output directory (e.g. /user_defined_output_directory/<omics>/<tool>/)
- Downstream clusterProfiler (or BioTranslator) enrichment runs on features that passed correlation-based filtering, demonstrating noise mitigation took effect
- Correlation coefficients fall within the expected range [-1, 1] and matrices are symmetric for pairwise comparisons between omics layers
- The Pearson-vs-Spearman choice is recorded in the module parameters, and the analysis is performed after (not before) the MCIA integration step
- Feature-pair connections flagged as low-information are absent from the feature set handed to pathway enrichment

## Limitations

- The article does not specify a numeric correlation cutoff or a default choice between Pearson and Spearman — thresholds are user-decided
- Correlation here is a noise-mitigation/filtering step feeding pathway enrichment; it does not replace the data-driven MCIA integration that reports the sample-space PCA and pseudo-eigen values
- Software versions for the individual analysis modules (including the correlation step) are not specified in the methods

## Evidence

- [intro] Correlation-based filtering of connections before pathway enrichment: "Correlation analysis performed with Pearson or Spearman can filter illuminating connection and patterns and aiding the noise mitigation for downstream analyses like pathway enrichment analysis"
- [intro] Enrichment after the MCIA integration step: "Clusterprofiler can be utilized by individual analyses or after the integration step of MCIA."
- [readme] Pipeline scope: "**multiOmicsIntegrator** is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
