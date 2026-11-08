---
name: kegg-pathway-topology-retrieval
description: Use when a multi-omics (gene + metabolite) pathway analysis needs the
  underlying KEGG pathway graph topology — node types, compound/gene identifiers,
  and gene–metabolite interactions — as input, e.g. to run eSPIA in the SMART 2.0
  IOPA module.
when_to_use_negative:
- The analysis is plain over-representation analysis (ORA) on gene or metabolite sets
  alone — ORA does not need pathway interaction topology, and on the SMART 2.0 breast
  cancer example it detected 51 significant pathways (pFDR < 0.05) versus 2 for eSPIA,
  so the two are not interchangeable
- The pathway database of interest is not KEGG — SMART 2.0 also references Reactome
  and WikiPathways, and this skill covers only KEGG gene/metabolite topology retrieval
- Only untargeted metabolomics peak preprocessing is needed (SMART modules 1–6); topology
  retrieval is specific to the integrative omics pathway analysis (IOPA) module
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: SMART 2.0
  role: Consumes the retrieved KEGG pathway topologies for its integrative omics pathway
    analysis (IOPA) module, running eSPIA on joint gene-and-metabolite pathways
  repo: https://github.com/YuJenL/SMART
- name: KEGGREST/KEGGgraph
  role: R packages used to retrieve KEGG metabolism pathway structures (gene and metabolite
    nodes plus their interaction topology)
- name: KEGG
  role: Pathway database providing pathway structures and compound/gene identifiers
    used for node mapping
- name: Pbine
  role: P-value combination method (with Fisher's method) applied to gene- and metabolite-level
    pathway evidence after topology-based impact analysis, with FDR/Bonferroni correction
  repo: https://github.com/YuJenL/SMART
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:42ba6e9cd4546e03baaa4b4bae96aba02ec650490a838a63d416bede002a0129
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# kegg-pathway-topology-retrieval

## Summary

Retrieves KEGG metabolism pathway structures — gene and metabolite (compound) nodes plus their interaction topology — via KEGGREST/KEGGgraph in R so that features from a metabolomics/omics study can be mapped onto pathway nodes for joint (integromics) pathway analyses such as eSPIA/IOPA, as implemented in SMART 2.0.

## When to use

Use when a multi-omics (gene + metabolite) pathway analysis needs the underlying KEGG pathway graph topology — node types, compound/gene identifiers, and gene–metabolite interactions — as input, e.g. to run eSPIA in the SMART 2.0 IOPA module.

## When NOT to use

- The analysis is plain over-representation analysis (ORA) on gene or metabolite sets alone — ORA does not need pathway interaction topology, and on the SMART 2.0 breast cancer example it detected 51 significant pathways (pFDR < 0.05) versus 2 for eSPIA, so the two are not interchangeable
- The pathway database of interest is not KEGG — SMART 2.0 also references Reactome and WikiPathways, and this skill covers only KEGG gene/metabolite topology retrieval
- Only untargeted metabolomics peak preprocessing is needed (SMART modules 1–6); topology retrieval is specific to the integrative omics pathway analysis (IOPA) module

## Inputs

- List of KEGG pathway IDs of interest (e.g. KEGG metabolism pathways hsa04151, hsa00030)
- Gene and metabolite feature tables from the study (e.g. SMART 2.0 breast cancer gene and metabolite feature tables)
- KEGG compound/gene identifier mappings for the study features

## Outputs

- KEGG pathway graph structures with gene and metabolite (compound) nodes and interaction topology
- Feature-to-node mappings (KEGG gene/compound IDs) per pathway
- Input structures for eSPIA/IOPA perturbation-accumulation analysis and per-pathway p-value combination (Fisher's or Pbine method)

## How to apply

Using KEGGREST and/or KEGGgraph in R, retrieve the KEGG metabolism pathway structures used by SMART 2.0's integrative omics pathway analysis (IOPA), including both gene nodes and metabolite/compound nodes and the interaction topology connecting them. Map the study's gene and metabolite features onto pathway nodes via KEGG compound and gene identifiers (e.g. KEGG IDs such as C02630 for 2-hydroxyglutarate, and pathway IDs such as hsa04151 for the PI3K-Akt signaling pathway). The retrieved topology is what allows SPIA to be extended (eSPIA) to jointly consider genes and metabolites, computing a perturbation-accumulation statistic per pathway; p-values from gene and metabolite sides can then be combined with Fisher's method or the Pbine method (with FDR/Bonferroni correction). For example, in the SMART 2.0 breast cancer demonstration the PI3K-Akt pathway (hsa04151) was the most significant with an FDR-corrected p-value of 2.80 × 10–10 using Pbine, while the pentose phosphate pathway (hsa00030) showed gene and metabolite ORA p-values of 0.02202 and 0.02009 respectively. Validation is judged by whether the topology retrieval supports reproduction of reported significance calls, e.g. eSPIA identifying 2 significant pathways at pFDR < 0.05 on the breast cancer data versus 51 by ORA.

## Related tools

- **SMART 2.0** (Consumes the retrieved KEGG pathway topologies for its integrative omics pathway analysis (IOPA) module, running eSPIA on joint gene-and-metabolite pathways) — https://github.com/YuJenL/SMART
- **KEGGREST/KEGGgraph** (R packages used to retrieve KEGG metabolism pathway structures (gene and metabolite nodes plus their interaction topology))
- **KEGG** (Pathway database providing pathway structures and compound/gene identifiers used for node mapping)
- **Pbine** (P-value combination method (with Fisher's method) applied to gene- and metabolite-level pathway evidence after topology-based impact analysis, with FDR/Bonferroni correction) — https://github.com/YuJenL/SMART

## Examples

```
Rscript -e 'library(KEGGREST); pw <- keggGet(c("hsa04151","hsa00030"))'
```

## Evaluation signals

- Each retrieved pathway contains both gene nodes and compound (metabolite) nodes with interaction topology, not just a gene list — required for eSPIA's joint perturbation analysis
- All study features (genes and metabolites) can be mapped to KEGG IDs; unmapped features are accounted for (e.g. HMDB-only compounds may lack KEGG counterparts, as seen with heroin and nimetazepam)
- Known pathways resolve to correct KEGG IDs (e.g. hsa04151 = PI3K-Akt signaling pathway, hsa00030 = pentose phosphate pathway)
- Downstream IOPA reproduces published significance calls: eSPIA yields 2 significant pathways (pFDR < 0.05) on the breast cancer data versus 51 for ORA; Pbine gives hsa04151 an FDR-corrected p-value of 2.80 × 10–10
- Per-pathway outputs carry raw and FDR-corrected p-values plus combined p-values (Fisher's or Pbine method)

## Limitations

- Only KEGG pathways with both gene and metabolite node annotation can support joint gene–metabolite impact analysis; compounds absent from KEGG cannot be mapped (SMART 2.0 could not detect heroin and nimetazepam because HMDB does not include them, illustrating identifier-source limitations)
- Significant topological (eSPIA) and over-representation (ORA) results can diverge sharply (2 vs 51 pathways at pFDR < 0.05 on the breast cancer data), so retrieval-driven topology analysis should not be validated against ORA calls
- Further biological experiments are warranted to confirm the significance of other novel genes and metabolites that have not yet been documented, so topology-based significance calls remain hypotheses to validate

## Evidence

- [discussion] the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [discussion] the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance: "the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively"
- [intro] combines diverse biological data with established pathway databases, including KEGG, Reactome, and WikiPathways: "combines diverse biological data with established pathway databases, including KEGG, Reactome, and WikiPathways"
- [discussion] the oncometabolite 2-hydroxyglutarate (2HG) (KEGG ID C02630): "the oncometabolite 2-hydroxyglutarate (2HG) (KEGG ID C02630)"
- [readme] Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA).: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
