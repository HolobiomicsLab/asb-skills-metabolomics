---
name: two-group-statistical-testing-metabolomics
description: Use when a metabolomics abundance matrix (metabolites x samples) with
  two group labels must be converted into a significant-metabolite query set for downstream
  pathway enrichment analysis.
when_to_use_negative:
- Input is already a curated list of KEGG IDs/names or a pre-computed query set —
  pass it directly as inputMetabolites to enrichmet() instead of running run_de()
- The metabolomics matrix has more than two groups or repeated measures — run_de()
  is defined for a two-group control vs mutant comparison
- The goal is MetSEA on ranked metabolite lists without differential selection — fgsea-based
  MetSEA can use summary statistics directly rather than the filtered query set
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: EnrichMet
  role: R package providing run_de() differential analysis and the enrichmet() entry
    point that consumes da_results to derive the enrichment query set
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Runtime environment in which run_de() and the enrichment workflow execute
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:a484354a71a0338ec36b99340b2241e9f4da4b49c3a56af710f5bf1ca09a8055
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# two-group-statistical-testing-metabolomics

## Summary

Differential analysis stage (run_de) in the EnrichMet R package that computes per-metabolite fold change between two metabolomics groups (e.g. control vs mutant lung cancer) and applies Benjamini-Hochberg adjustment to select the significant-metabolite query set feeding Fisher's Exact Test pathway enrichment.

## When to use

Use when a metabolomics abundance matrix (metabolites x samples) with two group labels must be converted into a significant-metabolite query set for downstream pathway enrichment analysis.

## When NOT to use

- Input is already a curated list of KEGG IDs/names or a pre-computed query set — pass it directly as inputMetabolites to enrichmet() instead of running run_de()
- The metabolomics matrix has more than two groups or repeated measures — run_de() is defined for a two-group control vs mutant comparison
- The goal is MetSEA on ranked metabolite lists without differential selection — fgsea-based MetSEA can use summary statistics directly rather than the filtered query set

## Inputs

- Metabolomics abundance matrix (metabolites as rows, samples as columns) in example_data.csv format with control and mutant group labels
- Two group label strings identifying the comparison arms (e.g. 'TK-CMV' and 'K-CMV')
- Fold-change threshold (fc_threshold = 1) and adjusted p-value threshold (pval_threshold = 0.05)

## Outputs

- da_results object from run_de() containing per-metabolite fold change, p-values, and Benjamini-Hochberg adjusted p-values
- Filtered significant-metabolite query set (absolute fold change >= 1, adjusted p-value < 0.05) passed to Fisher's Exact Test pathway enrichment
- Optional volcano plot of differential analysis results

## How to apply

Load the metabolomics abundance matrix in example_data.csv format (metabolites as rows, samples as columns) carrying control and mutant lung cancer group labels, then call run_de() in R/EnrichMet with the two group names (e.g. 'TK-CMV' vs 'K-CMV'), fc_threshold = 1 and pval_threshold = 0.05. run_de() computes per-metabolite fold change between the two groups, computes group-comparison p-values, and applies Benjamini-Hochberg multiple-testing adjustment. Filter metabolites to those with absolute fold change >= 1 and adjusted p-value < 0.05 to form the significant-metabolite query set; optionally render a volcano plot of the results (Figure S1). Pass the returned da_results object to enrichmet() via da_results with inputMetabolites = NULL so the query set is derived automatically for Fisher's Exact Test pathway enrichment. Supplying summary statistics with met_id, pval or padj, and log2fc columns additionally enables MetSEA and a measured-metabolite background.

## Related tools

- **EnrichMet** (R package providing run_de() differential analysis and the enrichmet() entry point that consumes da_results to derive the enrichment query set) — https://github.com/biodatalab/enrichmet.git
- **R** (Runtime environment in which run_de() and the enrichment workflow execute)

## Examples

```
da_out <- run_de(metabolomics_mat, "TK-CMV", "K-CMV", fc_threshold = 1, pval_threshold = 0.05); results_da <- enrichmet(inputMetabolites = NULL, PathwayVsMetabolites = PathwayVsMetabolites, da_results = da_out, example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df)
```

## Evaluation signals

- The da_results object carries the columns expected by enrichmet(): per-metabolite fold change plus pval/padj and log2fc summary statistics
- Every metabolite in the derived query set satisfies absolute fold change >= 1 and adjusted p-value < 0.05
- enrichmet() called with inputMetabolites = NULL and da_results resolves a non-empty input_metabolites_used count
- Volcano plot (if generated) shows the same hit/set counts as the filtered query set, matching Figure S1
- Resulting pathway_enrichment_results are produced via Fisher's Exact Test with Benjamini-Hochberg adjusted p-values

## Limitations

- Thresholds are fixed defaults (absolute fold change >= 1, adjusted p < 0.05); stricter or laxer criteria require overriding fc_threshold/pval_threshold
- Annotations must be fetched at runtime from public APIs (KEGG REST, Reactome) and cached with BiocFileCache — offline use requires user-supplied Pathway/Metabolites tables
- Fisher's Exact Test bias from database compounds never observed can be reduced only if a measured background is explicitly supplied via extract_measured_kegg_ids(example_summary)

## Evidence

- [intro] Differential analysis threshold selection: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05) to apply Fisher’s Exact Test for pathway enrichment analysis"
- [readme] run_de usage and parameters: "run_de(
  metabolomics_mat,
  "TK-CMV",
  "K-CMV",
  fc_threshold = 1,
  pval_threshold = 0.05
)"
- [readme] da_results wiring into enrichmet: "inputMetabolites can be left `NULL`."
- [readme] Volcano plot option: "If differential analysis is run with `run_de()`, a volcano plot can also
be included."
- [readme] Summary statistics columns for MetSEA/background: "also provide
summary statistics (`example_data`) with the columns `met_id`, `pval` or
`padj`, and `log2fc`."
