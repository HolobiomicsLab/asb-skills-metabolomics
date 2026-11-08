---
name: count-matrix-statistical-testing
description: Use when you have a filtered, normalized, batch-corrected gene or isoform
  count matrix plus a phenotype file and need differential expression testing between
  conditions, or Salmon quant.sf files needing isoform switch analysis.
when_to_use_negative:
- The count matrix is still raw — it must first pass through MOI's filtering (filterByExp
  or cutoff), normalization (e.g. quantile), and batch correction (ComBat/SVA) modules
  before DE testing
- The goal is pathway enrichment or multi-omics integration rather than condition-wise
  differential expression — use clusterProfiler/BioTranslator or MCIA instead
- Input is quant.sf files for the gene-level matrix workflow — isoformSwitchAnalyzeR
  handles Salmon quantification separately from the edgeR/DESeq2/RankProd count-matrix
  module
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: edgeR
  role: One of three DE algorithms run on the normalized count matrix, selected via
    params.yml
- name: DESeq2
  role: Alternative DE algorithm run on the normalized count matrix
- name: RankProd
  role: Third DE algorithm option in the MOI feature-acquisition stage
- name: ISOformSwitchAnalyzeR
  role: Takes Salmon quant.sf files to perform isoform- and gene-level differential
    expression and detect genome-wide isoform switches
- name: MultiOmicsIntegrator (MOI)
  role: Nextflow pipeline whose feature-acquisition stage wraps the DE algorithms
    and isoform switching analysis
  repo: https://github.com/ASAGlab/moi
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:45625862f921f6208a852097d2d1e0f85d65f4c6fa9f41b6a9f1f084b1fb06ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - CPAT
  - DESeq2
  - ISOformSwitchAnalyzeR
  - MultiOmicsIntegrator (MOI)
  - Pfam
  - RankProd
  - edgeR
  - signalP
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Count-matrix statistical testing

## Summary

Differential expression analysis of a preprocessed RNA-seq count matrix using edgeR, DESeq2, or RankProduct, as implemented in the MultiOmicsIntegrator (MOI) feature-acquisition stage, optionally paired with isoformSwitchAnalyzeR on Salmon quant.sf files for isoform-level DE and isoform switch detection.

## When to use

Use when you have a filtered, normalized, batch-corrected gene or isoform count matrix plus a phenotype file and need differential expression testing between conditions, or Salmon quant.sf files needing isoform switch analysis.

## When NOT to use

- The count matrix is still raw — it must first pass through MOI's filtering (filterByExp or cutoff), normalization (e.g. quantile), and batch correction (ComBat/SVA) modules before DE testing
- The goal is pathway enrichment or multi-omics integration rather than condition-wise differential expression — use clusterProfiler/BioTranslator or MCIA instead
- Input is quant.sf files for the gene-level matrix workflow — isoformSwitchAnalyzeR handles Salmon quantification separately from the edgeR/DESeq2/RankProd count-matrix module

## Inputs

- Preprocessed (filtered, normalized, batch-corrected) count matrix
- Phenotype/metadata file (condition labels per sample)
- params.yml specifying the DE algorithm (edgeR, DESeq2, or RankProd) and in/out paths
- Salmon quant.sf quantification files (for the isoform switch sub-workflow)

## Outputs

- Per-module differential expression result text tables
- RData objects with intermediate DE results
- Isoform-level and gene-level differential expression results from isoformSwitchAnalyzeR
- Genome-wide isoform switch calls with functional annotations (CPAT coding potential, Pfam protein domains, signalP signal sequences)

## How to apply

Load the normalized count matrix and phenotype file into the MOI differential expression module, selecting the algorithm (edgeR, DESeq2, or RankProd) and in/out paths through the module's params.yml file. Run the chosen method on the normalized matrix — which should already have gone through MOI preprocessing (filtering e.g. filterByExp or a cutoff, quantile normalization, and ComBat/SVA batch correction) — and write per-module intermediate results as text tables and RData objects. In parallel, feed Salmon quant.sf files into isoformSwitchAnalyzeR to perform differential expression at both the isoform and gene levels and detect genome-wide isoform switches; transcripts of interest are then annotated for coding potential with CPAT, protein-domain homology with Pfam, and signal sequences with signalP. Collect DE results and isoform switch annotations into the output directory for downstream pathway enrichment (clusterProfiler or BioTranslator) and inspection.

## Related tools

- **edgeR** (One of three DE algorithms run on the normalized count matrix, selected via params.yml)
- **DESeq2** (Alternative DE algorithm run on the normalized count matrix)
- **RankProd** (Third DE algorithm option in the MOI feature-acquisition stage)
- **ISOformSwitchAnalyzeR** (Takes Salmon quant.sf files to perform isoform- and gene-level differential expression and detect genome-wide isoform switches)
- **MultiOmicsIntegrator (MOI)** (Nextflow pipeline whose feature-acquisition stage wraps the DE algorithms and isoform switching analysis) — https://github.com/ASAGlab/moi

## Examples

```
nextflow run nf-core/mom -c parmas_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- The count matrix supplied to the DE module has already been filtered (filterByExp or cutoff), normalized (e.g. quantile via calcNorm), and batch-corrected (com/sva/comsva/svacom/none setting)
- The chosen algorithm name and in/out paths are correctly set in the module's params.yml
- Both text tables and RData objects are written per module as intermediate results
- isoformSwitchAnalyzeR consumes the Salmon quant.sf files and returns differential expression at both isoform and gene levels plus genome-wide isoform switch calls
- Transcripts of interest carry annotations for coding potential (CPAT), protein domains (Pfam), and signal sequences (signalP)

## Limitations

- Exact software versions for the DE tools are not specified in the methods table
- Only three DE algorithms are provided (edgeR, DESeq2, RankProduct); the choice is fixed via params.yml
- The isoform switch workflow depends on Salmon quant.sf files, so it cannot run without upstream Salmon quantification

## Evidence

- [intro] DE algorithms run on preprocessed matrices: "Now,is time to perform differential expression analysis. We provide three different algorithms for that, which we describe below."
- [intro] Preprocessing precedes DE testing: "the user can perform [filtering](../modules/local/mom_filter), [normalization](../modules/local/mom_norm/) and [batch effect](../modules/local/mom_filter/) correction"
- [intro] isoformSwitchAnalyzeR input and scope: "takes these quantification files and performs differential expression analysis on the level of both isoforms and genes"
- [intro] Functional annotation of switch transcripts: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline purpose from README: "**multiOmicsIntegrator** is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
