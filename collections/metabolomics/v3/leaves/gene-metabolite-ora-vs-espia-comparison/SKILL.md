---
name: gene-metabolite-ora-vs-espia-comparison
description: 'Use when you have both gene and metabolite feature tables mapped to
  KEGG pathway nodes and need to compare topology-aware impact analysis (eSPIA) against
  ORA, or to reproduce SMART 2.0''s IOPA breast cancer result (eSPIA: 2 significant
  pathways vs ORA: 51 at pFDR < 0.05).'
when_to_use_negative:
- Only a single omics type (genes OR metabolites alone) is available — the original
  SPIA/ORA single-type setting, not the joint gene-and-metabolite eSPIA analysis.
- Pathways lack interaction topology (e.g., non-KEGG sources without node/edge structure)
  — ORA alone is applicable, but eSPIA's perturbation-accumulation statistic requires
  KEGG pathway topology.
- The goal is untargeted peak processing (preprocessing, normalization, QC) rather
  than downstream pathway significance testing — use SMART 2.0's earlier modules instead.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3517
tools:
- name: SMART 2.0
  role: Provides the IOPA module implementing eSPIA (joint gene-and-metabolite pathway
    impact analysis) and ORA comparison, plus example breast cancer feature tables
  repo: https://github.com/YuJenL/SMART
- name: KEGGREST/KEGGgraph
  role: Retrieve KEGG metabolism pathway structures (gene and metabolite nodes plus
    interaction topology) in R
- name: Pbine
  role: Combine gene and metabolite pathway p-values via Fisher's method or numerical-integration
    Pbine method with FDR/Bonferroni correction
  repo: https://github.com/Yinchun-Lin/Pbine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:8c88ba7eb13673450f509485f2a462d3281e2843b055e481a82cb758a3081fbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - KEGGREST/KEGGgraph
  - Pbine
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gene-metabolite-ora-vs-espia-comparison

## Summary

Contrast topology-aware integrative pathway analysis (eSPIA, an impact-analysis extension of SPIA that jointly perturbs genes and metabolites in KEGG pathways) with over-representation analysis (ORA) on the same gene and metabolite feature sets, since the two methods can disagree sharply in how many pathways they call significant.

## When to use

Use when you have both gene and metabolite feature tables mapped to KEGG pathway nodes and need to compare topology-aware impact analysis (eSPIA) against ORA, or to reproduce SMART 2.0's IOPA breast cancer result (eSPIA: 2 significant pathways vs ORA: 51 at pFDR < 0.05).

## When NOT to use

- Only a single omics type (genes OR metabolites alone) is available — the original SPIA/ORA single-type setting, not the joint gene-and-metabolite eSPIA analysis.
- Pathways lack interaction topology (e.g., non-KEGG sources without node/edge structure) — ORA alone is applicable, but eSPIA's perturbation-accumulation statistic requires KEGG pathway topology.
- The goal is untargeted peak processing (preprocessing, normalization, QC) rather than downstream pathway significance testing — use SMART 2.0's earlier modules instead.

## Inputs

- breast cancer metabolite feature table (SMART package example data)
- breast cancer gene feature table (SMART package example data)
- KEGG metabolism pathway topology (gene and metabolite nodes with interactions) via KEGGREST/KEGGgraph
- KEGG compound and gene identifiers for mapping features onto pathway nodes

## Outputs

- espia_results.csv with per-pathway ID, impact statistic, raw and FDR-corrected p-value, and ORA comparison flag
- count of significant pathways under eSPIA (2 at pFDR < 0.05 on breast cancer data)
- count of significant pathways under ORA (51 at pFDR < 0.05 on breast cancer data)

## How to apply

Load the breast cancer metabolite and gene feature tables from the SMART package example data, then retrieve KEGG metabolism pathway structures (gene and metabolite nodes plus interaction topology) via KEGGREST/KEGGgraph in R and map features onto pathway nodes using KEGG compound/gene identifiers. Run eSPIA, which computes each pathway's perturbation-accumulation statistic and p-value while jointly considering genes and metabolites, then apply FDR correction across pathways and select significant pathways at pFDR < 0.05. Run ORA independently on the same gene and metabolite sets for comparison; in SMART 2.0's IOPA of the breast cancer data, eSPIA identified 2 significant pathways while ORA detected 51 at pFDR < 0.05. Where p-values must be integrated across gene and metabolite tests, use Fisher's method or the Pbine method (numerical integration of combined p-values, with FDR or Bonferroni correction) as implemented in the Yinchun-Lin/Pbine package. Write per-pathway results (pathway ID, impact statistic, raw and FDR-corrected p-value, ORA comparison flag) to espia_results.csv; judge success by reproducing the eSPIA-vs-ORA significance contrast on the breast cancer data.

## Related tools

- **SMART 2.0** (Provides the IOPA module implementing eSPIA (joint gene-and-metabolite pathway impact analysis) and ORA comparison, plus example breast cancer feature tables) — https://github.com/YuJenL/SMART
- **KEGGREST/KEGGgraph** (Retrieve KEGG metabolism pathway structures (gene and metabolite nodes plus interaction topology) in R)
- **Pbine** (Combine gene and metabolite pathway p-values via Fisher's method or numerical-integration Pbine method with FDR/Bonferroni correction) — https://github.com/Yinchun-Lin/Pbine

## Examples

```
Rscript -e 'library(SMART); # IOPA: load breast cancer gene/metabolite tables, run eSPIA via KEGGREST topology, FDR-correct, compare with ORA; write.csv(espia_results, "espia_results.csv")'
```

## Evaluation signals

- eSPIA identifies 2 significant pathways (pFDR < 0.05) on the SMART 2.0 breast cancer example data, matching the reported result
- ORA on the same gene and metabolite sets identifies 51 significant pathways (pFDR < 0.05), so the eSPIA-vs-ORA count contrast (2 vs 51) is reproduced
- Every pathway row in espia_results.csv carries a pathway ID, impact statistic, raw p-value, FDR-corrected p-value, and ORA comparison flag
- p-value combination of gene and metabolite tests uses Fisher's method or Pbine with FDR or Bonferroni correction, and corrected values remain within [0, 1]
- Features map onto KEGG pathway nodes only via valid KEGG compound/gene identifiers; unmapped features are excluded before statistic computation

## Limitations

- eSPIA is restricted to KEGG pathways with gene-and-metabolite topology; the article notes Reactome and WikiPathways exist but the IOPA demonstration uses KEGG metabolism pathways
- eSPIA and ORA can disagree strongly (2 vs 51 significant pathways), so significance counts are method-dependent and should not be treated as equivalent
- Pathway-level findings for undocumented genes and metabolites require further biological experiments to confirm significance (e.g., PI3K-Akt hsa04151 via Pbine, pentose phosphate pathway hsa00030 via gene/metabolite ORA p-values of 0.02202 and 0.02009)

## Evidence

- [results] eSPIA identified two significant pathways (pFDR < 0.05), whereas ORA detected 51 significant pathways (pFDR < 0.05): [evidence span withheld — anchor status: not_found]
- [methods] the combined p-values were integrated using Fisher's method or the Pbine method: [evidence span withheld — anchor status: not_found]
- [intro] This knowledge gap has driven us to develop a novel multiomics pathway analysis approach that combines diverse biological data with established pathway databases: "combines diverse biological data with established pathway databases, including KEGG, Reactome, and WikiPathways"
- [discussion] the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [discussion] the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively: "the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively"
