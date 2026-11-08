---
name: cross-omics-identifier-mapping
description: Use when merging or enriching multi-omics results (e.g. isoforms, miRNAs,
  proteins, lipids from MOI Salmon/isoformSwitchAnalyzeR outputs) that must be converted
  to gene-level features before pathway analysis.
when_to_use_negative:
- Input features are already gene-level mRNAs only with no miRNA, isoform, protein,
  or lipid layers to reconcile — no translation is needed
- Downstream analysis does not require pathway enrichment or cross-layer integration
  (e.g. layer-specific QC like FASTQC/Trim Galore)
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0092
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Provides the pipeline environment in which identifier translation to gene-level
    occurs across gene, isoform, miRNA, protein, and lipid layers before enrichment.
  repo: https://github.com/ASAGlab/moi
- name: clusterProfiler
  role: Consumes the gene-level translated feature set for over-representation pathway
    enrichment after cross-omics mapping.
- name: BioTranslator
  role: Alternative ORA tool that operates on the unified gene-level feature set produced
    by cross-omics identifier mapping.
- name: Salmon
  role: Produces quant.sf files whose transcript/isoform identifiers are among those
    translated to gene-level identifiers.
  repo: https://github.com/nf-core/modules
- name: isoformSwitchAnalyzeR
  role: Generates isoform-level differential expression results that are subsequently
    mapped to the gene namespace.
  repo: https://github.com/nf-core/modules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:db4eecf8c5f91565ded290162288ad46f3dbffd2c9591e99934f5a27b1070a64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ASAGlab/moi
  tools:
  - BioTranslator
  - MultiOmicsIntegrator (MOI)
  - Salmon
  - clusterProfiler
  - isoformSwitchAnalyzeR
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-omics-identifier-mapping

## Summary

Translates heterogeneous omics feature identifiers (miRNAs, isoforms, proteins, lipids, mRNAs) into a unified gene-level namespace so that downstream pathway enrichment and integration operate on comparable features.

## When to use

Use when merging or enriching multi-omics results (e.g. isoforms, miRNAs, proteins, lipids from MOI Salmon/isoformSwitchAnalyzeR outputs) that must be converted to gene-level features before pathway analysis.

## When NOT to use

- Input features are already gene-level mRNAs only with no miRNA, isoform, protein, or lipid layers to reconcile — no translation is needed
- Downstream analysis does not require pathway enrichment or cross-layer integration (e.g. layer-specific QC like FASTQC/Trim Galore)

## Inputs

- Differential expression results table per omics layer (genes, miRNAs, isoforms, proteins, lipids) from the MOI pipeline
- Phenotype/sample metadata file associated with each DE table
- Salmon quant.sf quantification files (for isoform/gene-level features)

## Outputs

- Gene-level feature table with a unified identifier namespace across omics layers
- Set of prioritized genes derived from all available omics layers, ready for clusterProfiler/BioTranslator enrichment
- Optional correlation-filtered feature connection list (Pearson or Spearman) used as noise mitigation before enrichment

## How to apply

After differential expression analysis within the MOI pipeline (e.g. Salmon quant.sf-based isoform/gene DE, miRNA DE, proteomics or lipidomics results), load each layer's differential expression table and translate non-gene feature identifiers to gene-level identifiers so all features share a common namespace. The article's genes module specifically routes miRNA, isoform, protein and lipid features through gene-level translation prior to pathway enrichment so 'all data can be incorporated into pathway enrichment analysis', yielding a set of prioritized genes derived from all available omics layers. Optionally apply Pearson or Spearman correlation-based filtering on feature connections to mitigate noise before enrichment. This translation step must occur before clusterProfiler or BioTranslator ORA, because those tools operate on gene sets. Validate that every input feature class (mRNAs, miRNAs, isoforms including lncRNAs, proteins, lipids) is represented in the translated gene set and that the enriched pathway list corresponds to the input differential-expression feature set.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Provides the pipeline environment in which identifier translation to gene-level occurs across gene, isoform, miRNA, protein, and lipid layers before enrichment.) — https://github.com/ASAGlab/moi
- **clusterProfiler** (Consumes the gene-level translated feature set for over-representation pathway enrichment after cross-omics mapping.)
- **BioTranslator** (Alternative ORA tool that operates on the unified gene-level feature set produced by cross-omics identifier mapping.)
- **Salmon** (Produces quant.sf files whose transcript/isoform identifiers are among those translated to gene-level identifiers.) — https://github.com/nf-core/modules
- **isoformSwitchAnalyzeR** (Generates isoform-level differential expression results that are subsequently mapped to the gene namespace.) — https://github.com/nf-core/modules

## Examples

```
nextflow run nf-core/mom -c params_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Every input omics layer (mRNAs, miRNAs, isoforms including lncRNAs, proteins, lipids) is represented in the translated gene-level output table
- The enriched pathway list corresponds to the input differential-expression feature set after translation
- Correlation filtering (Pearson or Spearman) applied to feature connections yields a reduced, noise-mitigated connection set before enrichment
- Gene-level table retains sample/phenotype alignment from the original DE tables
- Downstream clusterProfiler or BioTranslator run executes without identifier-missing errors

## Limitations

- Identifier mapping depends on availability of complete gene-level annotations for non-coding and non-mRNA feature classes; missing annotations can cause dropped features
- The article does not report quantitative accuracy of the mapping itself (e.g. one-to-many/many-to-one rates)
- Cross-omics translation is positioned before enrichment; applying enrichment on un-translated layers would be incorrect
- The MOI changelog's Added/Fixed sections are empty for the initial release, so known mapping-related fixes are undocumented

## Evidence

- [results] MOI's final stage first translates all available omics layers to the gene level so all data can be incorporated into pathway enrichment analysis, yielding a set of prioritized genes derived from all available omics layers.: [evidence span withheld — anchor status: not_found]
- [intro] Correlation analysis performed with Pearson or Spearman can filter illuminating connection and patterns and aiding the noise mitigation for downstream analyses like pathway enrichment analysis: "Correlation analysis performed with Pearson or Spearman can filter illuminating connection and patterns and aiding the noise mitigation for downstream analyses like pathway enrichment analysis"
- [intro] Last step of the analysis is to perform pathway enrichment analysis with clusterprofiler or biotranslator: "Last step of the analysis is to perform pathway enrichment analysis with [clusterprofiler](../modules/local/clusterprofiler) or biotranslator [biotranslator]"
- [intro] The pipeline performs RNAseq analysis on the level of mRNAs, miRNAs, and isoforms including lncRNAs, plus lipidomics and proteomics, and integration of multi omics data: "1. RNAseq analysis on the level of :
   - mRNAs 
   - miRNAs
   - isoforms including lncRNAs"
- [readme] **multiOmicsIntegrator** is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data.: "**multiOmicsIntegrator** is a bioinformatics best-practice analysis pipeline for analysis of multi-Omics data."
