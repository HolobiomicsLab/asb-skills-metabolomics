---
name: omics-feature-to-gene-translation
description: Use when downstream pathway enrichment (BioTranslator or clusterProfiler
  ORA) requires gene-level features but the input differential-expression tables come
  from miRNA, isoform, protein, or lipid analyses whose identifiers are not genes.
when_to_use_negative:
- Input is already a gene-level differential expression table — translation is unnecessary;
  run clusterProfiler ORA directly.
- The omics layer has no mapping to gene-level identifiers (e.g. lipids without annotated
  target genes) — no common gene namespace can be established.
- No pathway enrichment is planned downstream; translation only makes sense to feed
  BioTranslator or clusterProfiler ORA.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MultiOmicsIntegrator (MOI)
  role: Pipeline whose DE module produces the differential-expression tables and phenotype
    files that this translation step consumes, and whose final stage translates all
    omics layers to the gene level.
  repo: https://github.com/nf-core/modules
- name: BioTranslator
  role: Executed on the translated gene-level set to perform over-representation pathway
    enrichment; enrichment statistics and plots are exported to the genes/biotranslator/
    directory.
- name: clusterProfiler
  role: Alternative over-representation analysis tool applied to the translated gene
    set, either on individual analyses or after the MCIA integration step.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae175
    title: 'MultiOmicsIntegrator: a nextflow pipeline for integrated omics analyses'
  source_content_hash: sha256:3b81dc65c31fc03928611b2ff25631651e02988f3de6d0c538a5b8494d7579b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/modules
  tools:
  - BioTranslator
  - MultiOmicsIntegrator (MOI)
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# omics-feature-to-gene-translation

## Summary

Translates non-gene feature identifiers (miRNAs, isoforms, proteins, lipids) from upstream MOI differential-expression results into a common gene-level namespace so that all omics layers can feed a single pathway enrichment analysis and yield one prioritized gene set.

## When to use

Use when downstream pathway enrichment (BioTranslator or clusterProfiler ORA) requires gene-level features but the input differential-expression tables come from miRNA, isoform, protein, or lipid analyses whose identifiers are not genes.

## When NOT to use

- Input is already a gene-level differential expression table — translation is unnecessary; run clusterProfiler ORA directly.
- The omics layer has no mapping to gene-level identifiers (e.g. lipids without annotated target genes) — no common gene namespace can be established.
- No pathway enrichment is planned downstream; translation only makes sense to feed BioTranslator or clusterProfiler ORA.

## Inputs

- Differential expression results table for the omics layer of interest (from MOI DE module)
- Phenotype file for the omics layer of interest
- Non-gene feature identifiers: miRNAs, isoforms of genes (including lncRNAs), proteins, lipids

## Outputs

- Gene-level translated feature set derived from all available omics layers (prioritized genes)
- Enrichment statistics and plots exported as text and RData objects under /user_defined_output_directory/genes/biotranslator/
- Prioritized gene list corresponding to the input differential-expression feature set

## How to apply

Load the differential expression results table and phenotype file for the omics layer of interest produced upstream by MOI's DE module (e.g. edgeR, DESeq2, or RankProduct output). Translate non-gene feature identifiers — miRNAs, isoforms of genes including lncRNAs, proteins, and lipids — to gene-level identifiers so all features share a common namespace. Optionally apply Pearson or Spearman correlation-based filtering on feature connections to mitigate noise before enrichment. Run over-representation pathway enrichment on the translated gene set with BioTranslator (alternatively clusterProfiler's ORA) and export the enrichment statistics, plots, and intermediate results as text and RData objects under /user_defined_output_directory/genes/biotranslator/.

## Related tools

- **MultiOmicsIntegrator (MOI)** (Pipeline whose DE module produces the differential-expression tables and phenotype files that this translation step consumes, and whose final stage translates all omics layers to the gene level.) — https://github.com/nf-core/modules
- **BioTranslator** (Executed on the translated gene-level set to perform over-representation pathway enrichment; enrichment statistics and plots are exported to the genes/biotranslator/ directory.)
- **clusterProfiler** (Alternative over-representation analysis tool applied to the translated gene set, either on individual analyses or after the MCIA integration step.)

## Examples

```
nextflow run nf-core/mom -c params_file nf-core/mom/params_isoforms.yml -profile docker
```

## Evaluation signals

- Enrichment outputs exist in the genes/biotranslator/ directory in both text and RData formats.
- The enriched pathway list corresponds to the input differential-expression feature set.
- All omics layers are represented: the final gene set is derived from all available omics layers, not just the RNA-seq genes.
- Non-gene identifiers (miRNAs, isoforms, proteins, lipids) no longer appear in the translated feature set.
- If correlation filtering was applied, the retained connections should reflect Pearson or Spearman filtering on the feature connections.

## Limitations

- Correlation-based filtering (Pearson or Spearman) is optional noise mitigation; without it, noisy connections may propagate into enrichment results.
- Translation depends on upstream DE results being available for each omics layer; layers without DE output cannot contribute.
- Lipidomics features may not map cleanly to genes, limiting their contribution to the gene-level set.

## Evidence

- [other] MOI's final stage first translates all available omics layers to the gene level so all data can be incorporated into pathway enrichment analysis, yielding a set of prioritized genes derived from all available omics layers.: [evidence span withheld — anchor status: not_found]
- [intro] Last step of the analysis is to perform pathway enrichment analysis with clusterprofiler or biotranslator: "Last step of the analysis is to perform pathway enrichment analysis with [clusterprofiler](../modules/local/clusterprofiler) or biotranslator [biotranslator]"
- [abstract] MOI performs in-depth individual analyses of diverse omics including exhaustive RNA-seq analysis at gene, isoform and miRNA levels with functional annotation and structure prediction, supports proteomics and metabolomics, and integrates multiple omics datasets with data- and function-driven approaches: "MOI represents a unified approach that performs in-depth individual analyses of diverse omics. Specifically, exhaustive analysis of RNA-seq data at the level of genes, isoforms of genes, as well as"
- [intro] Clusterprofiler can be utilized by individual analyses or after the integration step of MCIA.: "Clusterprofiler can be utilized by individual analyses or after the integration step of MCIA."
