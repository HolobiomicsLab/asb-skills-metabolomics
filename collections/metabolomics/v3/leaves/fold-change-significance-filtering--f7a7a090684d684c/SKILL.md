---
name: fold-change-significance-filtering
description: Use when a metabolomics abundance matrix has been differentially analyzed
  between two groups (e.g. control vs mutant lung cancer) and you need a significant-metabolite
  list as input to pathway enrichment.
when_to_use_negative:
- Input is already a defined query set of metabolites (e.g. a hand-curated list of
  KEGG IDs) and no differential analysis was performed — pass it directly via inputMetabolites
  instead
- The goal is MetSEA on a full ranked metabolite list rather than over-representation
  on a selected significant subset, since MetSEA uses summary statistics like log2fc
  and p-values rather than a filtered query set
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: EnrichMet
  role: Provides run_de() to compute fold change, p-values, and Benjamini-Hochberg
    adjustment and to apply the |FC| >= 1 / padj < 0.05 filter before enrichment
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Implementation language in which run_de() and the enrichment workflow execute
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:a8e1411c4e3c9147bbd8aab3be2986f925cc98d35abbee802668edbf9e4359dd
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# fold-change-significance-filtering

## Summary

Selects a significant-metabolite query set from a metabolomics differential analysis by filtering on absolute fold change ≥ 1 and Benjamini-Hochberg adjusted p-value < 0.05; these filtered metabolites then feed Fisher's Exact Test pathway enrichment in EnrichMet.

## When to use

Use when a metabolomics abundance matrix has been differentially analyzed between two groups (e.g. control vs mutant lung cancer) and you need a significant-metabolite list as input to pathway enrichment.

## When NOT to use

- Input is already a defined query set of metabolites (e.g. a hand-curated list of KEGG IDs) and no differential analysis was performed — pass it directly via inputMetabolites instead
- The goal is MetSEA on a full ranked metabolite list rather than over-representation on a selected significant subset, since MetSEA uses summary statistics like log2fc and p-values rather than a filtered query set

## Inputs

- Metabolomics abundance matrix in example_data.csv format (metabolites as rows, samples as columns) with two-group labels (e.g. TK-CMV vs K-CMV)
- Differential analysis results from run_de() containing fold change, p-value, and adjusted p-value per metabolite

## Outputs

- Significant-metabolite query set (list of KEGG IDs or metabolite names) satisfying |fold change| >= 1 and adjusted p-value < 0.05
- da_results object suitable for passing to enrichmet() via da_results argument
- Optional volcano plot of differential analysis results

## How to apply

Run the differential analysis stage (e.g. EnrichMet's run_de()) on a metabolites-by-samples abundance matrix such as example_data.csv, carrying control and mutant lung cancer group labels, to compute per-metabolite fold change, group-comparison p-values, and Benjamini-Hochberg adjusted p-values. Filter metabolites to those meeting absolute fold change ≥ 1 and adjusted p-value < 0.05 to form the significant-metabolite query set; in EnrichMet this filtering is configured via the fc_threshold and pval_threshold arguments of run_de(). The filtered set is then passed to enrichmet() either directly via the da_results argument (leaving inputMetabolites = NULL) or as a character vector of KEGG IDs under input_type = 'kegg'. The rationale is that enrichment tests like Fisher's Exact Test need a defined query set, and the article establishes |FC| ≥ 1 and padj < 0.05 as the criteria for 'significant metabolites'. Optionally render a volcano plot of the differential results (Figure S1) to visualize the filtered set. Verify the filter by checking that the resulting da_results output contains only metabolites meeting both thresholds before downstream enrichment.

## Related tools

- **EnrichMet** (Provides run_de() to compute fold change, p-values, and Benjamini-Hochberg adjustment and to apply the |FC| >= 1 / padj < 0.05 filter before enrichment) — https://github.com/biodatalab/enrichmet.git
- **R** (Implementation language in which run_de() and the enrichment workflow execute)

## Examples

```
da_out <- run_de(metabolomics_mat, "TK-CMV", "K-CMV", fc_threshold = 1, pval_threshold = 0.05); results_da <- enrichmet(inputMetabolites = NULL, da_results = da_out, PathwayVsMetabolites = PathwayVsMetabolites)
```

## Evaluation signals

- Every metabolite in the filtered set satisfies absolute fold change >= 1 (the article's threshold)
- Every metabolite in the filtered set satisfies adjusted p-value < 0.05 after Benjamini-Hochberg correction
- The da_results object passed to enrichmet() contains the expected columns and yields a non-empty query set derived from the differential analysis
- Volcano plot (when generated) reflects the fold-change / adjusted-p distribution of the unfiltered differential results
- Downstream Fisher's Exact Test enrichment receives the filtered set as its query rather than the full metabolite list

## Limitations

- Thresholds of absolute fold change >= 1 and adjusted p < 0.05 are the article's chosen criteria; other cutoffs (e.g. stricter FC or FDR) may be required for datasets with different effect-size or power characteristics
- The filter requires a two-group comparison; it does not define query-set selection for single-group or multi-factor designs
- Volcano plotting of differential results is only available when run_de() is used; no volcano is generated if the query set is supplied directly as inputMetabolites

## Evidence

- [intro] Significant metabolite selection criteria: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [intro] Filtering feeds Fisher's Exact Test enrichment: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05) to apply Fisher’s Exact Test for pathway enrichment analysis"
- [readme] run_de() parameters and enrichment input in README: "run_de(
  metabolomics_mat,
  "TK-CMV",
  "K-CMV",
  fc_threshold = 1,
  pval_threshold = 0.05
)"
- [readme] Differential analysis feeding enrichmet(): "For an end-to-end analysis that starts from the metabolomics matrix
(metabolites as rows, samples as columns), run differential analysis
with `run_de()` and pass the result to `enrichmet()` via"
- [intro] Case study dataset groups: "This study used a dataset generated from metabolomic profiles comparing control and mutant lung cancer"
