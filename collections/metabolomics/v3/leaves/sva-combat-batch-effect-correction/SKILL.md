---
name: sva-combat-batch-effect-correction
description: Use when a raw gene/miRNA/isoform count matrix from an RNA-seq workflow
  (e.g. Salmon quantification) shows batch structure — samples processed in multiple
  batches or confounded runs — and must be batch-corrected before differential expression
  analysis.
when_to_use_negative:
- The dataset consists of samples processed in a single batch with no confounding
  — mom_batch_method_genes should be 'none' rather than applying ComBat/SVA unnecessarily.
- The matrix is raw and unfiltered — correction should follow MOI's filtering (filterByExp
  or cutoff) and quantile normalization stages, not replace them.
- The goal is single-omics transcript-level analysis outside MOI (e.g. nf-core/rnaseq)
  where batch correction is handled by that pipeline's own modules.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Hosts the standalone preprocess_matrix module exposing batch-effect correction
    via mom_batch_method_genes
  repo: https://github.com/ASAGlab/moi
- name: sva
  role: R package implementing surrogate variable analysis, selectable as 'sva', 'comsva',
    or 'svacom'
- name: limma
  role: R package supporting ComBat batch removal within the correction step
- name: edgeR
  role: R package used in the preprocessing stack (with calcNorm quantile normalization
    preceding correction)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:a6c1d5d81b6097f7c858b12a51afd48369824fa88ec38a0d1481ca055682d647
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - MultiOmicsIntegrator (MOI)
  - edgeR
  - limma
  - sva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sva-combat-batch-effect-correction

## Summary

Corrects batch effects in a filtered, normalized count matrix within the MultiOmicsIntegrator (MOI) Nextflow pipeline, using ComBat and/or Surrogate Variable Analysis (SVA) from the sva, limma, and edgeR R packages as configured by the mom_batch_method_genes parameter.

## When to use

Use when a raw gene/miRNA/isoform count matrix from an RNA-seq workflow (e.g. Salmon quantification) shows batch structure — samples processed in multiple batches or confounded runs — and must be batch-corrected before differential expression analysis.

## When NOT to use

- The dataset consists of samples processed in a single batch with no confounding — mom_batch_method_genes should be 'none' rather than applying ComBat/SVA unnecessarily.
- The matrix is raw and unfiltered — correction should follow MOI's filtering (filterByExp or cutoff) and quantile normalization stages, not replace them.
- The goal is single-omics transcript-level analysis outside MOI (e.g. nf-core/rnaseq) where batch correction is handled by that pipeline's own modules.

## Inputs

- Filtered, normalized count matrix (feature matrix) from MOI's mom_filter/mom_norm preprocessing stage
- Phenotype file with sample metadata including batch and factor columns
- params.yml with mom_batch_method_genes setting (com | sva | comsva | svacom | none)

## Outputs

- Batch-corrected count matrix exported as a text table
- Batch-corrected matrix as an RData object for downstream DESeq2/edgeR modules

## How to apply

In MOI's standalone pre-processing module, batch-effect correction runs as the third stage, after filtering (mom_filt_method_genes, e.g. filterByExp or a cutoff value) and normalization (mom_norm_method_genes = quantile via calcNorm), and consumes the user-supplied count matrix plus a phenotype file. Set mom_batch_method_genes in params.yml to one of: 'com' for ComBat, 'sva', 'comsva' for ComBat followed by sva, 'svacom' for sva followed by ComBat, or 'none' to skip. The correction is implemented with the sva, limma, and edgeR R packages, with factor columns defining batch and phenotype groups taken from the phenotype file. The corrected matrix is exported as both a text table and an RData object under the user-defined output directory so downstream modules (e.g. DESeq2/edgeR differential expression) can consume it. Choosing 'comsva' vs 'svacom' controls the ordering — SVA estimates surrogate variables that ComBat can then adjust for, or vice versa — so the order should match the study's confounding structure.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Hosts the standalone preprocess_matrix module exposing batch-effect correction via mom_batch_method_genes) — https://github.com/ASAGlab/moi
- **sva** (R package implementing surrogate variable analysis, selectable as 'sva', 'comsva', or 'svacom')
- **limma** (R package supporting ComBat batch removal within the correction step)
- **edgeR** (R package used in the preprocessing stack (with calcNorm quantile normalization preceding correction))

## Examples

```
mom_batch_method_genes = "com"  # com for combat, sva, comsva for combat & sva, svacom for sva and combat, none
```

## Evaluation signals

- Output RData object and text table both exist in the user-defined output directory after the run.
- Corrected matrix contains all input samples — no samples dropped relative to the input count matrix.
- The corrected values reflect the configured method: setting mom_batch_method_genes = 'com' must produce ComBat-adjusted values, 'svacom' sva-then-ComBat, etc.
- Downstream DESeq2/edgeR modules accept the RData output without format errors, confirming schema compatibility.
- With mom_batch_method_genes = 'none', the exported matrix equals the quantile-normalized (post-filter) matrix, isolating the correction step's effect.

## Limitations

- Software versions for the sva/limma/edgeR stack are not specified in the article's methods table (listed only as 'Genes, miRNA, isoforms | Quality control | FastQC, trimgalore' etc.).
- Correction ordering matters ('comsva' = combat & sva vs 'svacom' = sva and combat) but the article gives no guidance on which order suits which confounding structure.
- Batch correction is applied after normalization in a fixed sequence; it cannot be interleaved differently within the MOI module.

## Evidence

- [intro] Filtering, normalization and batch correction stages exposed as module-level options: "the user can perform [filtering](../modules/local/mom_filter), [normalization](../modules/local/mom_norm/) and [batch effect](../modules/local/mom_filter/) correction"
- [intro] mom_batch_method_genes parameter enumerates ComBat/SVA combinations: "mom_batch_method_genes          = "com" # com for combat, sva,  comsva for combat & sva, svacom for sva and comba, none"
- [readme] MOI pipeline context in which the preprocessing module operates: "**multiOmicsIntegrator** is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
