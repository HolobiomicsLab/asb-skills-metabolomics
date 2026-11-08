---
name: empirical-correlation-network-analysis
description: Use when you have a metabolomics data matrix (e.g. plasma GC-TOF-MS peak
  values with BinBase identifiers for unknowns) and want data-driven metabolite-metabolite
  edges that do not depend on pathway knowledge or structural annotation.
when_to_use_negative:
- When you only want direct biochemical transformations (KEGG RPAIR substrate-product
  pairs) and do not want statistically inferred co-variation edges
- When the input is already a network edge list rather than a matrix of measured metabolite
  values per sample
- When few samples are available, since Spearman/Pearson correlation edges with FDR
  < 0.05 require adequate replicate measurements to be stable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaMapR
  role: Shiny/R application that calculates the Empirical Dependency Network (Spearman
    correlations) and exports the edge list and node attributes
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Programming language in which MetaMapR and the correlation calculations are
    implemented
  repo: https://github.com/dgrapov/MetaMapR
- name: Cytoscape
  role: Downstream import and visualization of the exported network with empirical
    and domain-knowledge variables mapped to node attributes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:81d7789177ce64a22064d9b643145d09dd50e55d678283e1a1356ce34490a5fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - Cytoscape
  - MetaMapR
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# empirical-correlation-network-analysis

## Summary

Builds an empirical dependency network among measured metabolites by correlating their abundance profiles (non-parametric Spearman, or Pearson/biweight) across samples and accepting edges at FDR p < 0.05, as implemented in MetaMapR.

## When to use

Use when you have a metabolomics data matrix (e.g. plasma GC-TOF-MS peak values with BinBase identifiers for unknowns) and want data-driven metabolite-metabolite edges that do not depend on pathway knowledge or structural annotation.

## When NOT to use

- When you only want direct biochemical transformations (KEGG RPAIR substrate-product pairs) and do not want statistically inferred co-variation edges
- When the input is already a network edge list rather than a matrix of measured metabolite values per sample
- When few samples are available, since Spearman/Pearson correlation edges with FDR < 0.05 require adequate replicate measurements to be stable

## Inputs

- Metabolite abundance matrix (samples x metabolites) as a delimited file, e.g. plasma GC-TOF-MS measurements with BinBase identifiers
- Sample class labels (e.g. T1D progressor vs non-progressor) for downstream node attribute mapping

## Outputs

- Edge-list CSV of accepted correlation edges (FDR p < 0.05)
- Node-attribute CSV mapping numeric node IDs to uploaded metabolite rows
- Optionally an SVG network graphic or a Cytoscape-importable network

## How to apply

Upload the metabolite value table into MetaMapR (a Shiny-based R application) and enable Empirical Dependency Networks, which per the article are calculated based on non-parametric Spearman correlations (alternatively parametric Pearson or biweight correlations) between measured metabolite values. Accept correlation edges at FDR p-values < 0.05, and combine them with biochemical (KEGG RPAIR), structural (PubChem Tanimoto > 0.7), and spectral (cosine correlation > 0.7) edge types if desired. On the network tab, decide whether to return all edge types or only unique edges, which follow a hierarchy of Biochemical > Structural > Mass Spectral > Correlation. Numeric node IDs map to uploaded data rows by default (row 1 = node 1); use the node attributes tab to verify this mapping or set a custom unique edge index under network > more options. Export the edge list and node attributes as .csv files for downstream import into Cytoscape, where empirical and domain-knowledge variables can be mapped onto nodes.

## Related tools

- **MetaMapR** (Shiny/R application that calculates the Empirical Dependency Network (Spearman correlations) and exports the edge list and node attributes) — https://github.com/dgrapov/MetaMapR
- **R** (Programming language in which MetaMapR and the correlation calculations are implemented) — https://github.com/dgrapov/MetaMapR
- **Cytoscape** (Downstream import and visualization of the exported network with empirical and domain-knowledge variables mapped to node attributes)

## Examples

```
cor(t(metabolite_values), method = "spearman", use = "pairwise.complete.obs")  # pairwise Spearman correlations feeding FDR p < 0.05 edge filtering in MetaMapR
```

## Evaluation signals

- All edges in the exported edge list carry FDR p-values below 0.05
- Numeric node identifiers in the edge list correspond one-to-one to rows of the uploaded metabolite table (verifiable in the node attributes tab)
- Exported edge-list and node-attribute files exist and parse as comma separated value files
- Combined network connects metabolites lacking structural or biochemical annotation (unknowns) via empirical edges, consistent with the article's T1D NOD mouse example of >470 plasma metabolites

## Limitations

- Correlation edges capture statistical co-variation, not direct biochemical causation; direct transformations must come from KEGG RPAIR biochemical networks
- When the unique edges option is selected, correlation edges are lower priority in the hierarchy (Biochemical > Structural > Mass Spectral > Correlation) and may be dropped
- KEGG database limitations on directionality of biochemical transformations can affect the combined network, though this concerns the complementary edge types

## Evidence
