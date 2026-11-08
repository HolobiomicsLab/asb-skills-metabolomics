---
name: metabolite-differential-abundance-analysis
description: Use when you have a metabolite-by-sample abundance matrix (e.g. example_data.csv
  with control and mutant lung cancer group labels) and need a statistically filtered
  significant-metabolite query set to drive Fisher's Exact Test pathway enrichment.
when_to_use_negative:
- You already have a defined metabolite query set as KEGG IDs/names — in that case
  supply inputMetabolites directly to enrichmet() instead of running run_de().
- Your goal is MetSEA on the full ranked metabolite list (log2 fold change and p-values)
  rather than over-representation of a filtered significant set — MetSEA uses summary
  statistics, not the thresholded query set.
- The experiment has more than two conditions or paired/repeated-measure designs —
  the documented workflow is a two-group control vs mutant comparison.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0266
tools:
- name: EnrichMet
  role: Provides run_de(), the differential analysis stage that computes fold change
    and BH-adjusted p-values and derives the significant-metabolite query set for
    enrichment
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Runtime language in which run_de() and the EnrichMet workflow are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:91dcdd868c2e40df7d46fd521acb94735be513d47a523fa9e99747073a53ccad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0266
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-differential-abundance-analysis
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolite-differential-abundance-analysis

## Summary

Two-group differential analysis of a metabolomics abundance matrix using EnrichMet's run_de() in R, computing per-metabolite fold change, group-comparison p-values with Benjamini-Hochberg adjustment, and filtering to significant metabolites (|fold change| >= 1, adjusted p < 0.05) that feed downstream Fisher's Exact Test pathway enrichment.

## When to use

Use when you have a metabolite-by-sample abundance matrix (e.g. example_data.csv with control and mutant lung cancer group labels) and need a statistically filtered significant-metabolite query set to drive Fisher's Exact Test pathway enrichment.

## When NOT to use

- You already have a defined metabolite query set as KEGG IDs/names — in that case supply inputMetabolites directly to enrichmet() instead of running run_de().
- Your goal is MetSEA on the full ranked metabolite list (log2 fold change and p-values) rather than over-representation of a filtered significant set — MetSEA uses summary statistics, not the thresholded query set.
- The experiment has more than two conditions or paired/repeated-measure designs — the documented workflow is a two-group control vs mutant comparison.

## Inputs

- Metabolomics abundance matrix in example_data.csv format (metabolites x samples) with control and mutant lung cancer group labels
- Two group identifiers (e.g. 'TK-CMV' mutant vs 'K-CMV' control)
- fc_threshold = 1 and pval_threshold = 0.05

## Outputs

- da_results object (S3 data.frame) with per-metabolite fold change, p-values, and Benjamini-Hochberg adjusted p-values, consumable by enrichmet() via the da_results argument
- Significant-metabolite query set filtered at absolute fold change >= 1 and adjusted p value < 0.05
- Optional volcano plot of the differential analysis results (Figure S1)

## How to apply

Load the metabolomics abundance matrix in example_data.csv format (metabolites as rows, samples as columns) carrying the two group labels (e.g. 'K-CMV' control vs 'TK-CMV' mutant lung cancer). Call run_de() with the matrix, the mutant and control group names, fc_threshold = 1 and pval_threshold = 0.05; run_de computes per-metabolite fold change between the groups, group-comparison p-values, and applies Benjamini-Hochberg multiple-testing adjustment. The filtered significant-metabolite set (absolute fold change >= 1 and adjusted p value < 0.05) becomes the query set for Fisher's Exact Test pathway enrichment when da_results is passed to enrichmet() with inputMetabolites = NULL. Optionally render a volcano plot of the differential analysis results (as shown in Figure S1). Verify correctness by checking that the returned da_results carry pval/padj and log2fc-style summary statistics and that only metabolites meeting both thresholds enter the enrichment query.

## Related tools

- **EnrichMet** (Provides run_de(), the differential analysis stage that computes fold change and BH-adjusted p-values and derives the significant-metabolite query set for enrichment) — https://github.com/biodatalab/enrichmet.git
- **R** (Runtime language in which run_de() and the EnrichMet workflow are executed)

## Examples

```
da_out <- run_de(metabolomics_mat, "TK-CMV", "K-CMV", fc_threshold = 1, pval_threshold = 0.05)
```

## Evaluation signals

- da_results contains per-metabolite fold change, raw p-values, and Benjamini-Hochberg adjusted p-values, with columns (met_id, pval/padj, log2fc) compatible with enrichmet()'s example_data summary-statistics interface.
- The selected query set contains only metabolites meeting BOTH absolute fold change >= 1 and adjusted p value < 0.05.
- Passing da_results to enrichmet() with inputMetabolites = NULL yields a non-empty metabolite query set used for Fisher's Exact Test.
- A volcano plot (optional) renders with fold change and adjusted p-value axes reflecting the same thresholds.
- Query-set size should be a strict subset of the input matrix rows (only significant metabolites retained).

## Limitations

- Thresholds are fixed by convention in the workflow (absolute fold change >= 1, adjusted p < 0.05); different cutoffs require re-deriving the query set.
- The documented use case is a two-group comparison (control vs mutant lung cancer); no multi-factor or paired design is described.
- No changelog was found for the package (section=discussion), so version behavior of run_de() is not documented.

## Evidence

- [intro] Significant metabolite thresholds feed Fisher's Exact Test enrichment: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05) to apply Fisher’s Exact Test for pathway enrichment analysis"
- [readme] run_de() produces the query set consumed by enrichmet() via da_results: [evidence span withheld — anchor status: external_doc]
- [readme] run_de() invocation with thresholds and group labels: "da_out <- run_de(
  metabolomics_mat,
  "TK-CMV",
  "K-CMV",
  fc_threshold = 1,
  pval_threshold = 0.05
)"
- [readme] Volcano plot is the differential-analysis visualization: "If differential analysis is run with `run_de()`, a volcano plot can also be included."
