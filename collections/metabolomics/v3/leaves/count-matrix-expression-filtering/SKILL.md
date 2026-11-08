---
name: count-matrix-expression-filtering
description: Use when a raw RNA-seq (or miRNA/isoform) count matrix plus phenotype
  file must be cleaned of low-expression features prior to normalization, batch correction,
  or differential expression (edgeR/DESeq2/RankProduct).
when_to_use_negative:
- Input is already a normalized or batch-corrected expression table rather than a
  raw count matrix — re-filtering would double-process the data.
- Analysis of proteomics, lipidomics, or metabolomics raw data where edgeR-style count-based
  filtering (filterByExp) is not applicable.
- Filtering is requested for downstream correlation-based pathway enrichment (the
  correlation-based connection filtering is a distinct, later step in the workflow).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Host pipeline providing the standalone preprocess_matrix module (mom_filter)
    that applies expression filtering to the count matrix via mom_filt_method_genes
  repo: https://github.com/ASAGlab/moi
- name: edgeR
  role: R package supplying the filterByExp expression-based filtering heuristic and
    calcNorm quantile normalization used before/after filtering
- name: limma
  role: R package used within the preprocessing module alongside sva/edgeR for the
    filter–normalize–batch-correct chain
- name: Nextflow
  role: Workflow engine executing the mom_filter process within nf-core/mom (version
    23.04.2.5870)
  repo: https://github.com/nf-core/configs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:961f1c27a761e59a90af678476560b21048b8bbb68056c9fa3d8e6136fc842e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - MultiOmicsIntegrator (MOI)
  - Nextflow
  - edgeR
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# count-matrix-expression-filtering

## Summary

Filters a raw gene/miRNA/isoform count matrix to retain only detectable features before normalization and batch-effect correction, using MOI's filterByExp heuristic or a user-chosen expression cutoff configured via mom_filt_method_genes.

## When to use

Use when a raw RNA-seq (or miRNA/isoform) count matrix plus phenotype file must be cleaned of low-expression features prior to normalization, batch correction, or differential expression (edgeR/DESeq2/RankProduct).

## When NOT to use

- Input is already a normalized or batch-corrected expression table rather than a raw count matrix — re-filtering would double-process the data.
- Analysis of proteomics, lipidomics, or metabolomics raw data where edgeR-style count-based filtering (filterByExp) is not applicable.
- Filtering is requested for downstream correlation-based pathway enrichment (the correlation-based connection filtering is a distinct, later step in the workflow).

## Inputs

- Raw gene/miRNA/isoform count matrix (feature matrix, text table)
- Phenotype file accompanying the count matrix
- params.yml settings: mom_filt_method_genes ('filterByExp' or numeric cutoff)

## Outputs

- Filtered count matrix (detectable features retained) as a text table
- Filtered count matrix serialized as an RData object for downstream modules (DESeq2/edgeR)

## How to apply

Read the user-supplied count matrix (feature matrix) and its accompanying phenotype file as specified in the pipeline params.yml for the preprocessing segment. Set mom_filt_method_genes to 'filterByExp' (edgeR's expression-based heuristic) or provide a chosen numeric cutoff value to retain only detectable features. Run the filtering through MOI's standalone preprocess_matrix module (modules/local/mom_filter) so the filtered matrix can be piped directly into quantile normalization (mom_norm_method_genes = 'quantile', calcNorm) and optionally ComBat/SVA batch correction (mom_batch_method_genes = 'com', 'sva', 'comsva', 'svacom', or 'none') using the sva/limma/edgeR R packages. The rationale is to remove noise/undetectable features early so downstream pathway enrichment and differential-expression results are not diluted. Filter, normalize, and batch-correct in that order: filtering precedes normalization, and batch correction is applied after normalization.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Host pipeline providing the standalone preprocess_matrix module (mom_filter) that applies expression filtering to the count matrix via mom_filt_method_genes) — https://github.com/ASAGlab/moi
- **edgeR** (R package supplying the filterByExp expression-based filtering heuristic and calcNorm quantile normalization used before/after filtering)
- **limma** (R package used within the preprocessing module alongside sva/edgeR for the filter–normalize–batch-correct chain)
- **Nextflow** (Workflow engine executing the mom_filter process within nf-core/mom (version 23.04.2.5870)) — https://github.com/nf-core/configs

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Output RData and text-table artifacts exist in the user-defined output directory after the mom_filter process completes.
- The filtered matrix retains all input samples (columns) — only features (rows) are removed by the filter.
- The set of retained features is consistent with the configured method: filterByExp output matches edgeR's heuristic on the input counts, or the numeric-cutoff output matches features exceeding that cutoff.
- The filtered matrix is compatible with the subsequent quantile-normalization step (mom_norm_method_genes = 'quantile') and reflects only the filtering step, with no normalization or batch correction applied prematurely.
- The module-level artifacts match the ones the broader pipeline records for the preprocessing segment.

## Limitations

- Software versions for individual preprocessing tools are not specified in the paper's methods table.
- The filtering step is only documented for gene/miRNA/isoform count matrices via the 'genes' params; behavior for other omics data types is not described.
- The preprocessing (filter/normalize/batch-correct) module is optional — pipelines that skip it must handle detectability thresholds elsewhere.

## Evidence

- [intro] Filter methods and parameters configured via params.yml: "mom_filt_method_genes           = "filterByExp"  # filterByExp or choose a cutoff value"
- [intro] Preprocessing workflow with local filter module: "the user can perform [filtering](../modules/local/mom_filter), [normalization](../modules/local/mom_norm/) and [batch effect](../modules/local/mom_filter/) correction"
- [intro] Batch-correction options that follow filtering: "mom_batch_method_genes          = "com" # com for combat, sva,  comsva for combat & sva, svacom for sva and comba, none"
- [readme] Pipeline engine and containerization context for running the module: "The pipeline is built using [Nextflow](https://www.nextflow.io) version 23.04.2.5870 (IMPORTANT), a workflow tool to run tasks across multiple compute infrastructures"
