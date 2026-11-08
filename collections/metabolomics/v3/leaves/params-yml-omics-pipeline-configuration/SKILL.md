---
name: params-yml-omics-pipeline-configuration
description: Use when a raw feature count matrix plus phenotype file must be filtered,
  normalized (e.g. quantile), and batch-corrected (ComBat/SVA) as a standalone pre-processing
  step before downstream omics analysis with MOI.
when_to_use_negative:
- Input is already a filtered, normalized, batch-corrected feature table — re-running
  filtering/normalization would double-process the data.
- The goal is upstream RNA-seq read processing (FASTQC, Trim Galore, Salmon quantification)
  rather than count-matrix pre-processing — use the MOI quantification workflow steps
  instead.
- No phenotype/batch information is available and mom_batch_method_genes = 'none'
  suffices with no normalization needs — the pre-processing module adds no value.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Pipeline providing the standalone preprocess_matrix module whose filtering,
    normalization, and batch-effect correction behaviour is driven by the params.yml
    settings
  repo: https://github.com/ASAGlab/moi
- name: Nextflow
  role: Workflow engine used to launch the pipeline (nextflow run nf-core/mom -c params_file
    ... -profile docker); version 23.04.2.5870 required
  repo: https://github.com/nf-core/modules
- name: edgeR
  role: R package used by the batch-correction/normalization step of the pre-processing
    module
- name: limma
  role: R package used by the batch-correction step of the pre-processing module
- name: sva
  role: R package providing SVA-based batch-effect correction selected via mom_batch_method_genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:7589ac81754cefbe3306ffd4483d30ea1a61bf657c626a0e3407270a922d514a
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# params-yml-omics-pipeline-configuration

## Summary

Configure and run the MultiOmicsIntegrator (MOI) Nextflow pipeline's pre-processing module (count-matrix filtering, normalization, batch-effect correction) by setting the mom_filt_method_genes, mom_norm_method_genes, and mom_batch_method_genes parameters in a params.yml file.

## When to use

Use when a raw feature count matrix plus phenotype file must be filtered, normalized (e.g. quantile), and batch-corrected (ComBat/SVA) as a standalone pre-processing step before downstream omics analysis with MOI.

## When NOT to use

- Input is already a filtered, normalized, batch-corrected feature table — re-running filtering/normalization would double-process the data.
- The goal is upstream RNA-seq read processing (FASTQC, Trim Galore, Salmon quantification) rather than count-matrix pre-processing — use the MOI quantification workflow steps instead.
- No phenotype/batch information is available and mom_batch_method_genes = 'none' suffices with no normalization needs — the pre-processing module adds no value.

## Inputs

- User-supplied raw count matrix (feature matrix)
- Phenotype file accompanying the count matrix
- params.yml specifying mom_filt_method_genes, mom_norm_method_genes, mom_batch_method_genes and output directory

## Outputs

- Preprocessed count matrix exported as a text table
- Preprocessed matrix as an RData object for downstream modules (e.g. DESeq2/edgeR)

## How to apply

Write a params.yml that points to the user-supplied count matrix (feature matrix) and its accompanying phenotype file, then set mom_filt_method_genes to 'filterByExp' or a chosen cutoff value to retain only detectable features. Set mom_norm_method_genes = 'quantile' to apply calcNorm quantile normalization on the filtered matrix. Choose mom_batch_method_genes among 'com' (ComBat), 'sva', 'comsva' (ComBat & sva), 'svacom' (sva and ComBat), or 'none', relying on the sva/limma/edgeR R packages for correction. Launch the pipeline with Nextflow (e.g. via nf-core/mom with a -c params file and a container profile such as docker/singularity, as MOI uses Docker/Singularity containers). Validation: the exported RData object and text table must exist in the user-defined output directory, contain all input samples, and reflect the configured filter, normalization, and batch-correction methods so downstream modules (e.g. DESeq2/edgeR differential expression) can consume them.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Pipeline providing the standalone preprocess_matrix module whose filtering, normalization, and batch-effect correction behaviour is driven by the params.yml settings) — https://github.com/ASAGlab/moi
- **Nextflow** (Workflow engine used to launch the pipeline (nextflow run nf-core/mom -c params_file ... -profile docker); version 23.04.2.5870 required) — https://github.com/nf-core/modules
- **edgeR** (R package used by the batch-correction/normalization step of the pre-processing module)
- **limma** (R package used by the batch-correction step of the pre-processing module)
- **sva** (R package providing SVA-based batch-effect correction selected via mom_batch_method_genes)

## Examples

```
nextflow run nf-core/mom -c params_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Output RData object and text table both exist in the user-defined output directory.
- Preprocessed outputs contain all input samples (no samples dropped by the module).
- Applied methods in the outputs match the configured params: filterByExp or cutoff, calcNorm quantile normalization, and the chosen ComBat/SVA combination.
- Downstream modules (e.g. DESeq2/edgeR differential expression) can consume the exported RData/text artifacts without schema errors.

## Limitations

- Software versions for individual pre-processing tools are not specified in the article's methods.
- Only the listed batch-method keywords ('com', 'sva', 'comsva', 'svacom', 'none') and the 'filterByExp' or cutoff filtering options are supported by the params settings.
- ComBat/SVA correction depends on the accompanying phenotype file providing appropriate batch/factor columns set in params.

## Evidence

- [intro] mom_filt_method_genes = "filterByExp"  # filterByExp or choose a cutoff value: "mom_filt_method_genes           = "filterByExp"  # filterByExp or choose a cutoff value"
- [intro] quantile normalization via calcNorm: "mom_norm_method_genes           = "quantile"     # calcNorm quantile"
- [intro] batch effect correction options: "mom_batch_method_genes          = "com" # com for combat, sva,  comsva for combat & sva, svacom for sva and comba, none"
- [intro] preprocessing step listing: "the user can perform [filtering](../modules/local/mom_filter), [normalization](../modules/local/mom_norm/) and [batch effect](../modules/local/mom_filter/) correction"
- [readme] container reproducibility: "It uses Docker/Singularity containers making installation trivial and results highly reproducible."
