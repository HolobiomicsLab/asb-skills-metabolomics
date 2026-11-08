---
name: metabolic-pathway-enrichment-visualization
description: Use when a list of significant metabolites (e.g. Log2FC > 1.5, adj-p
  < 0.05) from HFD or other treatment comparisons needs mapping to KEGG pathways to
  identify which metabolic functions are over-represented.
when_to_use_negative:
- The input is raw uncorrected metabolite intensities rather than a differential (Log2FC/adj-p)
  metabolite list — MSEA will under-detect pathways as seen with the raw-data metabolite
  list.
- The metabolite identifiers cannot be mapped to HMDB or KEGG compound IDs, since
  enrichment is performed against the predefined KEGG pathways database.
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0154
tools:
- name: dbnorm
  role: Produces the batch-corrected (e.g. ber) metabolomics data matrix on which
    the differential metabolite list feeding MSEA is computed
  repo: https://github.com/NBDZ/dbnorm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:0a4f35b19e963ef09c4558360b4771e72f3bcb0d0e4644c3f6fc84fb0264f709
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ConsensusPathDB
  - Human Metabolome Database (HMDB)
  - KEGG pathways database
  - dbnorm
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolic-pathway-enrichment-visualization

## Summary

Annotate a differentially regulated metabolite list from a batch-corrected LC–MS metabolomics dataset to functional pathways via metabolite set enrichment analysis (MSEA) against KEGG pathways, reporting pathways meeting an at-least-2-metabolite and q-value < 0.05 significance threshold.

## When to use

Use when a list of significant metabolites (e.g. Log2FC > 1.5, adj-p < 0.05) from HFD or other treatment comparisons needs mapping to KEGG pathways to identify which metabolic functions are over-represented.

## When NOT to use

- The input is raw uncorrected metabolite intensities rather than a differential (Log2FC/adj-p) metabolite list — MSEA will under-detect pathways as seen with the raw-data metabolite list.
- The metabolite identifiers cannot be mapped to HMDB or KEGG compound IDs, since enrichment is performed against the predefined KEGG pathways database.

## Inputs

- CSV table of significant metabolites with log2 fold change and adjusted p-values (Log2FC > 1.5, adj-p < 0.05)
- Metabolite identifiers mappable to HMDB / KEGG compound IDs
- KEGG pathways database (4289 compound IDs) mapping

## Outputs

- Annotated list of metabolites assigned to functional pathways
- Table of significantly dysregulated KEGG pathways with metabolite counts and q-values (< 0.05)
- Visualization of pathway-level enrichment of the metabolite list

## How to apply

Start from the differential metabolite list produced by a volcano plot differential analysis (Log2FC > 1.5, adj-p-value < 0.05) on batch-corrected data (e.g. ber-corrected via dbnorm). Annotate the significant metabolites and map the list of identifiers to predefined KEGG pathways (4289 compound IDs), previously searched and filtered against the Human Metabolome Database (HMDB). Run the metabolite set enrichment analysis (MSEA) to compute pathway over-representation, then select dysregulated pathways that contain at least two metabolites and reach a q-value < 0.05 after multiple-testing correction. The rationale is that batch-effect-corrected lists contain more pathway-covered metabolites, so lipid metabolism categories such as alpha-Linolenic acid metabolism and fatty acid biosynthesis become detectable only after correction (e.g. ber) removes drift across batches.

## Related tools

- **dbnorm** (Produces the batch-corrected (e.g. ber) metabolomics data matrix on which the differential metabolite list feeding MSEA is computed) — https://github.com/NBDZ/dbnorm

## Examples

```
MSEA(significant_metabolites, pathways="KEGG", q_value_threshold=0.05, min_metabolites=2)
```

## Evaluation signals

- Every selected pathway must be represented by at least two metabolites from the input list.
- All reported pathways must have q-value < 0.05 after multiple-testing correction.
- The metabolite list entering MSEA must be the one meeting Log2FC > 1.5 and adj-p-value < 0.05.
- Enriched pathways for an HFD-vs-control metabolite list should include lipid metabolism categories such as alpha-Linolenic acid metabolism and fatty acid biosynthesis when computed on ber-corrected data.
- Identifier mapping coverage: all input metabolites should be HMDB-searched/filter-able before KEGG mapping.

## Limitations

- Pathway enrichment is sensitive to batch effect correction choice: the raw-data metabolite list contained too few metabolites from lipid metabolism pathways to trigger enrichment, while ber-corrected data did.
- Coverage depends on how many metabolites from the input list can be mapped to KEGG compound IDs; pathways with < 2 mapped metabolites are excluded.
- The workflow assumes a predefined KEGG pathways database (4289 compound IDs); other pathway databases are not described here.

## Evidence

- [results] Differential analysis with MSEA on corrected data: "we next annotated the metabolites that were significantly influenced by HFD to functional pathways by using metabolite set enrichment analysis (MSEA)"
- [results] Threshold for significant metabolites: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [methods] Mapping identifiers to KEGG and HMDB filtering: "The list of identifiers was mapped to predefined KEGG pathways database enlisted by 4289 compound IDs."
- [results] Batch correction enables pathway detection: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
