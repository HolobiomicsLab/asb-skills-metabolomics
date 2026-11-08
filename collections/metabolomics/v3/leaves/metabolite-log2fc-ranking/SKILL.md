---
name: metabolite-log2fc-ranking
description: Use when you have metabolomics summary statistics (per-metabolite log2fc
  plus pval/padj columns) from a two-condition comparison and need a ranked metabolite
  list as input for fgsea-based metabolite set enrichment analysis (MetSEA).
when_to_use_negative:
- You only have an unordered list of significant metabolites with no log2fc or p-value
  statistics — over-representation analysis (Fisher's exact test) is the appropriate
  module instead of MetSEA
- The pathway map is not representable as GMT-compatible ranked-list annotations and
  cannot be converted from the curated KEGG pathway-metabolite associations
- Differential statistics have not been computed and condition labels/sample groups
  are not defined, so no log2fc or p-value ranking can be built
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3408
tools:
- name: EnrichMet
  role: R package whose MetSEA module (analysis_type = 'gsea') runs the enrichment
    on the ranked metabolite list and returns gsea_results with NES and adjusted p-values
  repo: https://github.com/biodatalab/enrichmet.git
- name: fgsea
  role: Underlying gene-set enrichment engine called by MetSEA on the log2fc/p-value
    ranked list against GMT-formatted pathway annotations
- name: KEGG
  role: Source of the curated pathway-metabolite associations that are converted to
    GMT format and used as the reference metabolite sets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:887b3787c9d23a6d74ffcf8598c6ce4c74d47ca84acb0d8bfc80b1e32aabfcc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  - fgsea
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3408
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-log2fc-ranking

## Summary

Rank metabolites from a control vs. condition metabolomics dataset by log2 fold change and p-value to build the ordered list that EnrichMet's MetSEA module feeds to fgsea, enabling metabolite set enrichment analysis against GMT-formatted KEGG pathway annotations.

## When to use

Use when you have metabolomics summary statistics (per-metabolite log2fc plus pval/padj columns) from a two-condition comparison and need a ranked metabolite list as input for fgsea-based metabolite set enrichment analysis (MetSEA).

## When NOT to use

- You only have an unordered list of significant metabolites with no log2fc or p-value statistics — over-representation analysis (Fisher's exact test) is the appropriate module instead of MetSEA
- The pathway map is not representable as GMT-compatible ranked-list annotations and cannot be converted from the curated KEGG pathway-metabolite associations
- Differential statistics have not been computed and condition labels/sample groups are not defined, so no log2fc or p-value ranking can be built

## Inputs

- Metabolomics summary statistics table with columns met_id, pval (or padj), and log2fc (e.g. summary_stat.csv from Zenodo record 17819145)
- Output of run_de() on a metabolomics matrix (metabolites as rows, samples as columns)
- GMT-formatted pathway annotations (KEGG pathway-metabolite associations, e.g. from fetch_kegg_pathway_metabolites(organism = 'hsa'))

## Outputs

- Ranked metabolite list ordered by log2 fold change and p-value
- fgsea/MetSEA results table (gsea_results) with pathway, pval, padj, and NES columns
- MetSEA plot (gsea_plot) of enriched pathways
- Set of significant pathways (adjusted p-value < 0.05) with directionality from NES sign

## How to apply

First run or obtain differential analysis of the metabolomics matrix (metabolites as rows, samples as columns), e.g. with EnrichMet's run_de() using fc_threshold = 1 and pval_threshold = 0.05, so each metabolite carries log2fc and pval/padj values. Order the metabolites by log2 fold change and p-value to form a ranked list; the article notes that pathway annotations are converted into Gene Matrix Transposed (GMT) format specifically to ensure compatibility with fgsea, so keep the pathway-to-metabolite map (e.g. KEGG hsa pathways fetched at runtime) in that format. Pass the ranked list and GMT annotations to the MetSEA module (analysis_type includes 'gsea' in the enrichmet() call), which invokes fgsea. Interpret results via the returned gsea_results table: a positive or negative NES with adjusted p-value < 0.05 marks a significantly enriched pathway. Rationale: ranking preserves directionality of metabolite change, allowing fgsea to detect coordinated up- or down-shifts across a pathway rather than membership alone.

## Related tools

- **EnrichMet** (R package whose MetSEA module (analysis_type = 'gsea') runs the enrichment on the ranked metabolite list and returns gsea_results with NES and adjusted p-values) — https://github.com/biodatalab/enrichmet.git
- **fgsea** (Underlying gene-set enrichment engine called by MetSEA on the log2fc/p-value ranked list against GMT-formatted pathway annotations)
- **KEGG** (Source of the curated pathway-metabolite associations that are converted to GMT format and used as the reference metabolite sets)

## Examples

```
da_out <- run_de(metabolomics_mat, "TK-CMV", "K-CMV", fc_threshold = 1, pval_threshold = 0.05); results <- enrichmet(da_results = da_out, PathwayVsMetabolites = PathwayVsMetabolites, analysis_type = c("enrichment", "gsea"))
```

## Evaluation signals

- The gsea_results table contains the expected fgsea columns: pathway, pval, padj, and NES
- Significant pathways are those with adjusted p-value < 0.05 (the paper's MetSEA criterion), each carrying a signed NES
- On the control vs. mutant lung cancer case study, recovered enriched categories include metabolic reprogramming and energy production, nutrient uptake and homeostasis (Figure 2C alignment)
- Every ranked metabolite maps to a KEGG identifier present in the GMT annotation sets; unannotated metabolites are flagged rather than silently dropped
- Ranking ordering is monotone in log2fc (with p-value as tie-breaker), matching the run_de output statistics

## Limitations

- Summary statistics must include the exact columns met_id, pval or padj, and log2fc, or MetSEA cannot be run
- Differential-analysis thresholds (fc_threshold = 1, pval_threshold = 0.05) directly shape the ranked list, so ranking quality depends on the underlying statistical model
- KEGG and Reactome annotations are fetched at runtime from public APIs and cached with BiocFileCache; the package does not redistribute database dumps, so availability of network/API access is required

## Evidence

- [intro] MetSEA using fgsea on ranked metabolite lists (log2 fold change and p-values) with GMT-formatted pathway annotations: [evidence span withheld — anchor status: not_found]
- [intro] GMT conversion for fgsea compatibility: "Pathway annotations are converted into a Gene Matrix Transposed (GMT) format to ensure compatibility with fgsea"
- [intro] MetSEA significance criterion: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
- [readme] Required summary-statistics columns for MetSEA: [evidence span withheld — anchor status: external_doc]
- [readme] run_de thresholds that feed the ranked list: "da_out <- run_de(
  metabolomics_mat,
  "TK-CMV",
  "K-CMV",
  fc_threshold = 1,
  pval_threshold = 0.05
)"
