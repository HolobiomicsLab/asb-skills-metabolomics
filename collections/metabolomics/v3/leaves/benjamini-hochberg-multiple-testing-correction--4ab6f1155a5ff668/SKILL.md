---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a metabolomics pathway-enrichment step yields one raw p-value
  per pathway (e.g. from Fisher's Exact Test on 2x2 contingency tables or fgsea MetSEA)
  and pathways must be ranked by adjusted p-value with a significance cutoff (adjusted
  p < 0.05).
when_to_use_negative:
- Testing a single pre-specified hypothesis (one pathway, no multiplicity) where no
  FDR correction is needed
- Applying to raw metabolite-level p-values before pathway aggregation — EnrichMet
  performs differential-analysis filtering first, then corrects at the pathway level
- When the user's downstream tool already supplies a corrected padj column (e.g. fgsea
  output) — double correction would over-shrink significance
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: EnrichMet
  role: Applies Benjamini-Hochberg adjustment inside the Fisher's Exact Test over-representation
    and MetSEA modules, ranking pathways by adjusted p-value
  repo: https://github.com/biodatalab/enrichmet.git
- name: fgsea
  role: Runs metabolite set enrichment analysis on ranked log2FC/p-value lists and
    returns per-pathway p-values that are Benjamini-Hochberg adjusted
- name: R
  role: Language in which the correction and downstream ranking/visualization are
    implemented
  repo: https://github.com/biodatalab/enrichmet.git
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:85623904c3c63d17abd910eabe62ba4ccade841a117096e634e8fedc1205ac7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  - fgsea
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Benjamini-Hochberg FDR correction applied to raw p-values from metabolomics pathway enrichment (Fisher's Exact Test over-representation, fgsea-based MetSEA) to control the false discovery rate across hundreds of KEGG pathways simultaneously.

## When to use

Use when a metabolomics pathway-enrichment step yields one raw p-value per pathway (e.g. from Fisher's Exact Test on 2x2 contingency tables or fgsea MetSEA) and pathways must be ranked by adjusted p-value with a significance cutoff (adjusted p < 0.05).

## When NOT to use

- Testing a single pre-specified hypothesis (one pathway, no multiplicity) where no FDR correction is needed
- Applying to raw metabolite-level p-values before pathway aggregation — EnrichMet performs differential-analysis filtering first, then corrects at the pathway level
- When the user's downstream tool already supplies a corrected padj column (e.g. fgsea output) — double correction would over-shrink significance

## Inputs

- Vector of raw pathway-level p-values from Fisher's Exact Test over-representation analysis (one per KEGG pathway)
- fgsea MetSEA results table with per-pathway pval, padj, NES columns
- Significant metabolite list (absolute fold change >= 1, adjusted p < 0.05) mapped to KEGG pathway membership (Pathway/Metabolites table)
- Optional measured-metabolite background (KEGG IDs from summary statistics) for background-corrected Fisher's exact test

## Outputs

- Pathway enrichment table with Adjusted_P_value column (e.g. results$pathway_enrichment_results with Pathway, P_value, Adjusted_P_value, Enrichment_Ratio)
- Ranked pathway list ordered by -log10(adjusted p-value) for dot plots and pathway impact plots
- MetSEA results with per-pathway padj and NES used to flag significant positive/negative enrichment
- Exportable CSV such as ora_pathway_enrichment_table.csv

## How to apply

After computing raw p-values for every pathway — e.g. Fisher's Exact Test applied to the 2x2 contingency table of significant metabolites (absolute fold change >= 1, adjusted p < 0.05) vs pathway membership from the KEGG PathwayVsMetabolites map — apply the Benjamini-Hochberg step-up procedure to the full vector of pathway p-values. EnrichMet applies this within its over-representation module and within the MetSEA module where fgsea returns per-pathway pval/padj/NES columns. Rank pathways by -log10(adjusted p-value) for dot-plot visualization, and treat pathways with adjusted p-value < 0.05 as significantly enriched (for MetSEA, a positive or negative NES accompanied by adjusted p < 0.05 indicates significant enrichment). When Fisher's exact test is restricted to a measured metabolite background via extract_measured_kegg_ids(), the correction is applied to the same pathway-level p-values so the FDR remains comparable across runs. In the lung cancer case study this produced pyrimidine metabolism as the top-ranked pathway by adjusted p-value.

## Related tools

- **EnrichMet** (Applies Benjamini-Hochberg adjustment inside the Fisher's Exact Test over-representation and MetSEA modules, ranking pathways by adjusted p-value) — https://github.com/biodatalab/enrichmet.git
- **fgsea** (Runs metabolite set enrichment analysis on ranked log2FC/p-value lists and returns per-pathway p-values that are Benjamini-Hochberg adjusted)
- **R** (Language in which the correction and downstream ranking/visualization are implemented) — https://github.com/biodatalab/enrichmet.git

## Examples

```
PathwayVsMetabolites <- fetch_kegg_pathway_metabolites(organism = "hsa"); results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df, p_value_cutoff = 1)
```

## Evaluation signals

- The enrichment table contains an Adjusted_P_value column and the top-ranked pathway (by that column) matches the article's reported result — pyrimidine metabolism for the lung cancer control-vs-mutant case study
- Number of pathways reported (results$pathway_enrichment_results rows) equals the count with P_value <= p_value_cutoff after BH adjustment, consistent with results$pathway_enrichment_all
- Every pathway retains both a raw P_value and an Adjusted_P_value, and Adjusted_P_value >= P_value monotonically under BH step-up ordering
- MetSEA padj values stay within [0,1] and the flagged significant pathways (padj < 0.05) show the expected sign of NES (positive or negative enrichment direction)
- Re-running with the measured background (backgroundMetabolites = extract_measured_kegg_ids(example_summary)) yields adjusted p-values comparable in order of magnitude to the uncorrected-background run

## Limitations

- BH adjustment is applied at the pathway level; it cannot rescue enrichment calls when too few metabolites (below min_metabolite_occurrence / min_pathway_occurrence) were queried
- EnrichMet does not redistribute KEGG/Reactome/LION dumps — annotations are fetched at runtime, so adjusted p-values can shift if the upstream pathway databases change between runs
- When pathways tested are few (e.g. 32 in the README example), BH is conservative relative to whole-transcriptome-scale corrections; the MetSEA example shows many padj values (0.3169) remaining far above the 0.05 cutoff

## Evidence

- [intro] Pathway enrichment analysis (PEA) / over-representation analysis via Fisher's Exact Test on a 2x2 contingency table with Benjamini-Hochberg adjustment: "Fisher’s Exact Test [12] was applied to the contingency table to assess whether the number of identified metabolites in a pathway significantly exceeded expectations"
- [intro] Significant metabolite filter feeding the contingency table: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [intro] MetSEA significance criterion combining NES and adjusted p-value: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
- [readme] fgsea MetSEA output columns include per-pathway adjusted p-values: [evidence span withheld — anchor status: not_found]
- [readme] EnrichMet enrichment table reports raw and adjusted p-values side by side: "results$pathway_enrichment_results[, c("Pathway", "P_value", "Adjusted_P_value", "Enrichment_Ratio")]"
- [other] Case study validation: pyrimidine metabolism top pathway by adjusted p-value: [evidence span withheld — anchor status: not_found]
