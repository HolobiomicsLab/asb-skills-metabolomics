---
name: pathway-feature-mapping-kegg-ids
description: Use when metabolomics or multiomics feature tables carry KEGG IDs and
  you need to place features on KEGG pathway gene/compound nodes (with topology) before
  ORA or perturbation impact analysis such as eSPIA.
when_to_use_negative:
- Features lack KEGG identifiers and cannot be reliably cross-referenced to KEGG compound/gene
  IDs
- The pathway of interest is absent from KEGG or from the metabolism category (e.g.,
  heroin and nimetazepam cannot be handled since HMDB does not encompass them and
  SMART 2.0 is unable to detect them)
- Only a single omics layer is analyzed and topology-aware joint gene–metabolite impact
  analysis (eSPIA/IOPA) is not the goal
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3345
- http://edamontology.org/topic_3174
tools:
- name: SMART 2.0
  role: R package whose IOPA module consumes the KEGG-mapped gene and metabolite features;
    example data and code at the SMART GitHub repo
  repo: https://github.com/YuJenL/SMART
- name: KEGGREST/KEGGgraph
  role: R libraries used to retrieve KEGG metabolism pathway structures (gene and
    metabolite nodes plus interaction topology)
- name: Pbine
  role: p-value combination (Fisher's / weighted Fisher's with numerical integration)
    applied to combined gene–metabolite pathway evidence after mapping
  repo: https://github.com/Yinchun-Lin/Pbine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:801833137b92e8f1775a5b9c0f3cac258dc36aa57529d66a3cd4998d3d964042
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - KEGG
  - KEGGREST/KEGGgraph
  - Pbine
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3345
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pathway-feature-mapping-kegg-ids

## Summary

Maps metabolite and gene features onto KEGG pathway nodes via KEGG compound/gene identifiers, retrieving gene-and-metabolite pathway topology with KEGGREST/KEGGgraph so topological methods like eSPIA/IOPA can be run in SMART 2.0.

## When to use

Use when metabolomics or multiomics feature tables carry KEGG IDs and you need to place features on KEGG pathway gene/compound nodes (with topology) before ORA or perturbation impact analysis such as eSPIA.

## When NOT to use

- Features lack KEGG identifiers and cannot be reliably cross-referenced to KEGG compound/gene IDs
- The pathway of interest is absent from KEGG or from the metabolism category (e.g., heroin and nimetazepam cannot be handled since HMDB does not encompass them and SMART 2.0 is unable to detect them)
- Only a single omics layer is analyzed and topology-aware joint gene–metabolite impact analysis (eSPIA/IOPA) is not the goal

## Inputs

- Metabolite feature table from SMART example data with KEGG compound IDs (e.g., C02630)
- Gene feature table with KEGG gene IDs (hsa*)
- KEGG metabolism pathway structures (gene/metabolite nodes and interaction topology) retrieved via KEGGREST/KEGGgraph

## Outputs

- Per-pathway feature-to-node mappings for genes and metabolites
- Feature-annotated pathway graphs used by ORA and eSPIA/IOPA
- Per-pathway test inputs (gene and metabolite lists) yielding p-values, e.g., pentose phosphate pathway ORA p-values of 0.02202 (gene) and 0.02009 (metabolite)

## How to apply

Load the metabolite and gene feature tables (e.g., the SMART breast cancer example data) and ensure each feature is annotated with a KEGG identifier (e.g., KEGG ID C02630 for 2-hydroxyglutarate, genes with hsa gene IDs). Retrieve KEGG metabolism pathway structures — gene and metabolite nodes plus their interaction topology — using KEGGREST/KEGGgraph in R, then match features to pathway nodes via KEGG compound/gene identifiers. Because pathway analysis in SMART 2.0 'combines diverse biological data with established pathway databases, including KEGG, Reactome, and WikiPathways', restrict mapping to pathways containing both mapped genes and metabolites when the downstream method is the integrative omics pathway analysis (IOPA) module. The mapped node assignments feed both ORA (per gene and per metabolite tests, judged at pFDR < 0.05) and eSPIA's perturbation-accumulation statistics with combined p-values (Fisher's method or the Pbine method).

## Related tools

- **SMART 2.0** (R package whose IOPA module consumes the KEGG-mapped gene and metabolite features; example data and code at the SMART GitHub repo) — https://github.com/YuJenL/SMART
- **KEGGREST/KEGGgraph** (R libraries used to retrieve KEGG metabolism pathway structures (gene and metabolite nodes plus interaction topology))
- **Pbine** (p-value combination (Fisher's / weighted Fisher's with numerical integration) applied to combined gene–metabolite pathway evidence after mapping) — https://github.com/Yinchun-Lin/Pbine

## Examples

```
library(KEGGREST); genes <- keggLink("hsa", "hsa04151"); cpds <- keggLink("compound", "hsa04151")
```

## Evaluation signals

- Every mapped feature resolves to a valid KEGG ID (e.g., 2-hydroxyglutarate maps to C02630) and a node in a retrieved KEGG pathway
- Reproducing the reported breast cancer result: eSPIA identifies 2 significant pathways at pFDR < 0.05, contrasting with ORA's 51 significant pathways
- Per-pathway ORA p-values are reported separately for genes and metabolites (e.g., hsa00030 pentose phosphate: p = 0.02202 gene, p = 0.02009 metabolite)
- espia_results.csv contains pathway ID, impact statistic, raw and FDR-corrected p-value, and ORA comparison flag for each pathway
- Significant topology findings reconcile with documented biology, e.g., PI3K-Akt signaling pathway (hsa04151) FDR-corrected p-value 2.80 × 10–10 using the Pbine method

## Limitations

- Some compounds are out of scope of reference databases: 'Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them'
- Novel significant genes and metabolites lack documentation; 'Further biological experiments are warranted to confirm the significance of other novel genes and metabolites that have not yet been documented'
- eSPIA can be far more conservative than ORA (2 vs. 51 significant pathways at pFDR < 0.05), so node mapping alone does not guarantee significance calls

## Evidence

- [intro] combines diverse biological data with established pathway databases, including KEGG, Reactome, and WikiPathways: "combines diverse biological data with established pathway databases, including KEGG, Reactome, and WikiPathways"
- [discussion] the oncometabolite 2-hydroxyglutarate (2HG) (KEGG ID C02630): "the oncometabolite 2-hydroxyglutarate (2HG) (KEGG ID C02630)"
- [discussion] the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively: "the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively"
- [readme] Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA).: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
