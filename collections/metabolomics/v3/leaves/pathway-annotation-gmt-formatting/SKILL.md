---
name: pathway-annotation-gmt-formatting
description: Use when a metabolite set enrichment analysis (MetSEA/GSEA) step requires
  pathway-to-metabolite annotations in a specific file format and the source annotations
  are only available as a table, KEGG dump, or API response.
when_to_use_negative:
- The input is already in GMT format or the downstream tool consumes pathway-metabolite
  mappings natively (e.g. a custom `Pathway`/`Metabolites` table is accepted directly),
  so reformatting would be redundant.
- The analysis goal is Fisher's exact-test over-representation on a metabolite set
  without ranked statistics; GMT conversion is not needed for that module.
- Annotations are for a non-metabolite feature type (e.g. genes) that requires a different
  identifier namespace than KEGG compound IDs.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0154
tools:
- name: EnrichMet
  role: Supplies the MetSEA module that consumes the GMT-formatted annotations together
    with the ranked metabolite list to run fgsea enrichment.
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Implementation language in which the GMT conversion and the fgsea-based MetSEA
    call are executed.
  repo: https://github.com/biodatalab/enrichmet.git
- name: KEGG
  role: Source of the curated pathway-metabolite associations that are converted into
    GMT format.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:f771e0bac291dc71c1018c2bbdba19334cb67a60a32ed3733b0c070543a5724a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Pathway annotation GMT formatting

## Summary

Converts curated pathway-metabolite associations (e.g. from KEGG) into Gene Matrix Transposed (GMT) format so ranked metabolite lists can be enriched with fgsea in the MetSEA module of EnrichMet.

## When to use

Use when a metabolite set enrichment analysis (MetSEA/GSEA) step requires pathway-to-metabolite annotations in a specific file format and the source annotations are only available as a table, KEGG dump, or API response.

## When NOT to use

- The input is already in GMT format or the downstream tool consumes pathway-metabolite mappings natively (e.g. a custom `Pathway`/`Metabolites` table is accepted directly), so reformatting would be redundant.
- The analysis goal is Fisher's exact-test over-representation on a metabolite set without ranked statistics; GMT conversion is not needed for that module.
- Annotations are for a non-metabolite feature type (e.g. genes) that requires a different identifier namespace than KEGG compound IDs.

## Inputs

- curated pathway-metabolite association table (e.g. KEGG `PathwayVsMetabolites` with organism code 'hsa')
- KEGG compound ID lookup (`kegg_lookup` from `fetch_kegg_compound_lookup()`) when input_type = 'name' or 'mixed'
- metabolite summary statistics with columns `met_id`, `pval`/`padj`, `log2fc` for ranking

## Outputs

- GMT-formatted pathway annotation file (pathway name, description, tab-separated KEGG compound IDs)
- MetSEA enrichment table with columns `pathway`, `pval`, `padj`, `NES` filtered to adjusted p-value < 0.05
- fgsea-compatible ranked metabolite list (ordered by log2 fold change and p-value)

## How to apply

Retrieve the pathway-to-metabolite association data (e.g. `fetch_kegg_pathway_metabolites(organism = "hsa")` for human KEGG pathways, cached locally with BiocFileCache), or supply your own `Pathway`/`Metabolites` table. Convert these annotations into a Gene Matrix Transposed (GMT) format, where each pathway is a row and the corresponding metabolite identifiers (KEGG compound IDs such as C00031 or C00022) are tab-separated columns; this is the format fgsea expects for its `fgsea()` call. Pass the GMT along with a metabolite list ranked by log2 fold change and p-value (or the `example_data` summary statistics containing `met_id`, `pval`/`padj`, and `log2fc` columns) into the MetSEA module. Filter the resulting enrichment table for pathways with adjusted p-value < 0.05, keeping the NES and adjusted-p columns. Validate by confirming the enrichment table has `pathway`, `pval`, `padj`, and `NES` fields and that significant pathways correspond to the expected biological categories (e.g. metabolic reprogramming; energy production, nutrient uptake and homeostasis).

## Related tools

- **EnrichMet** (Supplies the MetSEA module that consumes the GMT-formatted annotations together with the ranked metabolite list to run fgsea enrichment.) — https://github.com/biodatalab/enrichmet.git
- **R** (Implementation language in which the GMT conversion and the fgsea-based MetSEA call are executed.) — https://github.com/biodatalab/enrichmet.git
- **KEGG** (Source of the curated pathway-metabolite associations that are converted into GMT format.)

## Examples

```
PathwayVsMetabolites <- fetch_kegg_pathway_metabolites(organism = "hsa")
```

## Evaluation signals

- The produced GMT has one row per pathway with tab-separated metabolite identifiers that match the KEGG compound ID namespace used in the ranked list.
- Running fgsea on the GMT returns an enrichment table containing the columns `pathway`, `pval`, `padj`, and `NES` (not merely p-values alone).
- After filtering to adjusted p-value < 0.05, the remaining significant pathways map onto the expected biological categories reported in the paper (metabolic reprogramming; energy production, nutrient uptake and homeostasis).
- The ranked metabolite list preserves the original log2 fold change and p-value ordering from the differential analysis (absolute fold change >= 1 and adjusted p-value < 0.05).
- Cross-check that the number of metabolites per pathway in the GMT matches the count in the source `PathwayVsMetabolites` table before conversion.

## Limitations

- GMT conversion is specific to the ranked-list / fgsea (MetSEA) workflow; it is not required for Fisher's exact-test over-representation or the topology-based network analysis modules of EnrichMet.
- Pathway and compound annotations must be fetched at runtime from KEGG REST (or supplied by the user as an equivalent `Pathway`/`Metabolites` table); the package does not redistribute KEGG database dumps, so offline or stale caches may yield missing or outdated annotations.
- Significance is judged at adjusted p-value < 0.05; pathways above this cutoff are discarded from the reported MetSEA results, which may exclude borderline pathways.
- The ranked list must be derived from a differential analysis that already applied absolute fold change >= 1 and adjusted p-value < 0.05 to the input metabolites; feeding an unfiltered list changes the GMT-to-ranking correspondence.

## Evidence

- [intro] GMT conversion rationale: "Pathway annotations are converted into a Gene Matrix Transposed (GMT) format to ensure compatibility with fgsea"
- [intro] MetSEA ranking inputs: [evidence span withheld — anchor status: not_found]
- [intro] MetSEA significance threshold: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
- [readme] README annotation fetch: [evidence span withheld — anchor status: external_doc]
- [readme] README input requirements: [evidence span withheld — anchor status: external_doc]
- [readme] README output schema: [evidence span withheld — anchor status: not_found]
