---
name: multiple-testing-adjustment-metabolomics
description: Use when a metabolomics abundance matrix (metabolites x samples, control
  vs mutant groups) yields per-metabolite p-values, or when Fisher's Exact Test pathway
  hits must be judged against an adjusted p-value cutoff of 0.05.
when_to_use_negative:
- Input is already a pre-filtered list of significant metabolites with no raw per-metabolite
  p-values to adjust
- The analysis performs a single pre-registered hypothesis test rather than many simultaneous
  metabolite/pathway tests, so multiplicity correction is unwarranted
- The goal is a ranked-list GSEA-style scan where no fold-change/p-value cutoff filtering
  is desired before enrichment
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0602
tools:
- name: EnrichMet
  role: R package whose run_de() performs the group-comparison p-values with Benjamini-Hochberg
    adjustment and whose Fisher's Exact Test enrichment reports BH-adjusted pathway
    p-values
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Implementation language for run_de(), the BH adjustment, and the downstream
    EnrichMet workflow
- name: fgsea
  role: Runs MetSEA on ranked metabolite lists (log2 fold change and p-values), with
    significance judged by adjusted p-value < 0.05
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:2799284308708d645dc1bb41d12fce35785b6f974f1fc7167f9fcaffb6d0bc80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  - fgsea
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Multiple-testing adjustment for metabolomics differential analysis and enrichment

## Summary

Applies Benjamini-Hochberg multiple-testing correction to metabolomics group-comparison p-values and to Fisher's Exact Test pathway p-values, defining significance via adjusted p-value < 0.05. This controls false positives before selecting the significant-metabolite query set and before declaring enriched pathways.

## When to use

Use when a metabolomics abundance matrix (metabolites x samples, control vs mutant groups) yields per-metabolite p-values, or when Fisher's Exact Test pathway hits must be judged against an adjusted p-value cutoff of 0.05.

## When NOT to use

- Input is already a pre-filtered list of significant metabolites with no raw per-metabolite p-values to adjust
- The analysis performs a single pre-registered hypothesis test rather than many simultaneous metabolite/pathway tests, so multiplicity correction is unwarranted
- The goal is a ranked-list GSEA-style scan where no fold-change/p-value cutoff filtering is desired before enrichment

## Inputs

- Metabolomics abundance matrix in example_data.csv format (metabolites as rows, samples as columns) with control and mutant lung cancer group labels
- Per-metabolite raw p-values from the run_de() group comparison
- Per-pathway raw p-values from Fisher's Exact Test on the 2x2 contingency table
- fgsea/MetSEA results with pval, padj, and NES columns

## Outputs

- run_de() results table with Benjamini-Hochberg-adjusted p-values (padj) and log2fc per metabolite
- Significant-metabolite query set (absolute fold change >= 1 and adjusted p-value < 0.05) for Fisher's Exact Test enrichment
- BH-adjusted pathway p-values (Adjusted_P_value column in pathway_enrichment_results)
- MetSEA pathways flagged significant at adjusted p-value < 0.05 with NES sign

## How to apply

After run_de() computes per-metabolite fold changes and group-comparison p-values in R/EnrichMet, apply Benjamini-Hochberg adjustment to the raw p-values and filter metabolites to those with absolute fold change >= 1 and adjusted p-value < 0.05; only these form the query set passed to Fisher's Exact Test pathway enrichment. The same BH logic applies downstream: Fisher's Exact Test p-values on the 2x2 contingency table of pathway membership are BH-adjusted, and MetSEA results from fgsea are judged by adjusted p-value < 0.05 alongside positive or negative NES. The run_de() output table must carry padj (or pval) and log2fc columns so enrichmet() can derive the query set and the measured-metabolite background via extract_measured_kegg_ids(). Rationale: without BH correction across metabolites (hundreds of simultaneous tests) and across pathways, fold-change-only filtering would inflate the false-positive rate of both the metabolite selection and the pathway calls.

## Related tools

- **EnrichMet** (R package whose run_de() performs the group-comparison p-values with Benjamini-Hochberg adjustment and whose Fisher's Exact Test enrichment reports BH-adjusted pathway p-values) — https://github.com/biodatalab/enrichmet.git
- **R** (Implementation language for run_de(), the BH adjustment, and the downstream EnrichMet workflow)
- **fgsea** (Runs MetSEA on ranked metabolite lists (log2 fold change and p-values), with significance judged by adjusted p-value < 0.05)

## Examples

```
da_out <- run_de(metabolomics_mat, "TK-CMV", "K-CMV", fc_threshold = 1, pval_threshold = 0.05)
```

## Evaluation signals

- run_de() output table contains a padj (or pval) column plus log2fc for every metabolite, as required by enrichmet()
- No metabolite enters the Fisher's Exact Test query set unless it satisfies absolute fold change >= 1 AND adjusted p-value < 0.05 simultaneously
- Adjusted p-values are monotonically >= the corresponding raw p-values (sanity check of BH implementation)
- Reported enriched pathways (pathway_enrichment_results, e.g. 10 of 32 tested in the README example) each carry an Adjusted_P_value at or below the cutoff
- MetSEA calls flagged significant show |NES| directionality consistent with log2 fold-change ranks and padj < 0.05

## Limitations

- No changelog is documented for the package, limiting traceability of threshold or behavior changes
- BH adjustment is specific to the metabolite and pathway families tested here; other correction schemes (e.g. FDR variants or q-values) are not offered
- The adjusted p < 0.05 cutoff combined with absolute fold change >= 1 may yield an empty query set on low-power metabolomics datasets, stalling downstream enrichment

## Evidence

- [intro] We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05) to apply Fisher's Exact Test for pathway enrichment analysis: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05) to apply Fisher’s Exact Test for pathway enrichment analysis"
- [intro] A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
- [methods] Fisher's Exact Test was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations: "Fisher’s Exact Test [12] was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations"
- [readme] For MetSEA, or to build a measured-metabolite background, also provide summary statistics (example_data) with the columns met_id, pval or padj, and log2fc.: "For MetSEA, or to build a measured-metabolite background, also provide
summary statistics (`example_data`) with the columns `met_id`, `pval` or
`padj`, and `log2fc`."
- [readme] run_de call with fc_threshold = 1 and pval_threshold = 0.05: "da_out <- run_de(
  metabolomics_mat,
  "TK-CMV",
  "K-CMV",
  fc_threshold = 1,
  pval_threshold = 0.05
)"
