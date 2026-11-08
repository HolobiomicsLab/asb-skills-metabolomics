---
name: pathway-topology-analysis-espia
description: Use when a multiomics (gene expression + metabolomics) study needs pathway-level
  significance that accounts for pathway topology over KEGG, beyond simple over-representation
  of differentially abundant features.
when_to_use_negative:
- The analysis targets only narcotics identification or concentration calibration
  (SMART 2.0 peak identification and calibration modules), not pathway significance.
- Only one omics layer is available (metabolites alone or genes alone) and no topology-informed
  multiomics comparison is intended — plain ORA is the appropriate choice.
- The pathway database of interest is not KEGG (e.g., Reactome or WikiPathways); the
  demonstrated eSPIA workflow is over KEGG pathways.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3172
tools:
- name: SMART 2.0
  role: Hosts the IOPA module that runs eSPIA over KEGG pathway topology on combined
    gene-expression and metabolomics data
  repo: https://github.com/YuJenL/SMART
- name: Pbine
  role: Companion p-value combination tool (Fisher / weighted Fisher via numerical
    integration with a p-value correlation matrix sigma) used to integrate gene and
    metabolite pathway evidence alongside eSPIA
  repo: https://github.com/Yinchun-Lin/Pbine
- name: KEGG
  role: Pathway database supplying the pathway topology over which eSPIA significance
    is computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:b8fe5aaa90e36b436018552d8183b292e7e0bdfdf3d3302cabca2c2837f2b137
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - KEGG
  - Pbine
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pathway-topology-analysis-espia

## Summary

Extended significance analysis of pathway networks (eSPIA) within the SMART 2.0 integrative omics pathway analysis (IOPA) module, which tests KEGG pathway topology using combined gene-expression and metabolite evidence. In the SMART 2.0 breast cancer demonstration it detected two significant pathways where ORA alone detected 51.

## When to use

Use when a multiomics (gene expression + metabolomics) study needs pathway-level significance that accounts for pathway topology over KEGG, beyond simple over-representation of differentially abundant features.

## When NOT to use

- The analysis targets only narcotics identification or concentration calibration (SMART 2.0 peak identification and calibration modules), not pathway significance.
- Only one omics layer is available (metabolites alone or genes alone) and no topology-informed multiomics comparison is intended — plain ORA is the appropriate choice.
- The pathway database of interest is not KEGG (e.g., Reactome or WikiPathways); the demonstrated eSPIA workflow is over KEGG pathways.

## Inputs

- Gene-expression data matrix (e.g., breast cancer multiomics example dataset from YuJenL/SMART)
- Metabolomics feature data (untargeted or targeted metabolite abundances)
- KEGG pathway topology database
- Merged gene–metabolite KEGG ID mapping

## Outputs

- Table of significant pathways from eSPIA with pathway IDs, names, p-values, and significance flags (pFDR < 0.05)
- Combined ORA / eSPIA / Pbine pathway results table (e.g., smart_iopa_results.csv)

## How to apply

Run eSPIA as part of the SMART 2.0 IOPA module (Statistical Analysis module of the SMART package, YuJenL/SMART) after loading the gene-expression and metabolomics data. Supply KEGG pathway topology so that pathway structure, not only membership counts, informs significance, and evaluate results at pFDR < 0.05. Interpret eSPIA results alongside ORA (over-representation of genes/metabolites) and the Pbine p-value combination method (Fisher or weighted Fisher combination via numerical integration, accounting for p-value correlation with a sigma matrix) — in the breast cancer example eSPIA flagged 2 significant pathways versus 51 from ORA and 34 from Pbine. Compile pathway IDs, names, p-values, and significance flags for the three methods into a single table for comparison, recording top hits such as the PI3K-Akt signaling pathway (hsa04151, FDR-corrected p = 2.80e-10 via Pbine).

## Related tools

- **SMART 2.0** (Hosts the IOPA module that runs eSPIA over KEGG pathway topology on combined gene-expression and metabolomics data) — https://github.com/YuJenL/SMART
- **Pbine** (Companion p-value combination tool (Fisher / weighted Fisher via numerical integration with a p-value correlation matrix sigma) used to integrate gene and metabolite pathway evidence alongside eSPIA) — https://github.com/Yinchun-Lin/Pbine
- **KEGG** (Pathway database supplying the pathway topology over which eSPIA significance is computed)

## Evaluation signals

- Significance threshold respected: all reported pathways pass pFDR < 0.05.
- Reproduction check on the SMART 2.0 breast cancer example: eSPIA yields 2 significant pathways, ORA 51, and Pbine 34.
- Top Pbine hit is the PI3K-Akt signaling pathway (hsa04151) with FDR-corrected p-value 2.80e-10.
- Output table contains pathway IDs, names, p-values, and significance flags for ORA, eSPIA, and Pbine (e.g., smart_iopa_results.csv).
- Pbine run supplies (or estimates from paired p-value sequences) a correlation matrix sigma when combining p-values.

## Limitations

- eSPIA alone flagged far fewer pathways (2) than ORA (51) on the same breast cancer data, so it should be read jointly with ORA and Pbine rather than in isolation.
- Metabolite data alone may not fully elucidate the biological mechanisms underlying phenotypes, motivating integration with other omics layers; single-omics eSPIA is not validated here.
- Further biological experiments are warranted to confirm the significance of novel genes and metabolites not yet documented in the pathway results.

## Evidence

- [discussion] the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [abstract] SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [readme] Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA).: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
- [readme] This repository provides a tool for a meta-analysis based on a combination of p-values from multiple studies.: [evidence span withheld — anchor status: external_doc]
- [other] Applying IOPA to the breast cancer multiomics data, SMART 2.0 detected 51 significant pathways via ORA, two via eSPIA, and 34 via Pbine (all pFDR < 0.05).: [evidence span withheld — anchor status: not_found]
