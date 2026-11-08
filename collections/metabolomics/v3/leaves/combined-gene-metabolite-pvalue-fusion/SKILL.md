---
name: combined-gene-metabolite-pvalue-fusion
description: Use when two or more omics-specific p-value sequences (e.g., gene ORA
  and metabolite ORA over the same KEGG pathways) must be combined per pathway into
  a single FDR-corrected significance call.
when_to_use_negative:
- Only one omics evidence stream is available, so there is nothing to combine — run
  a single-stream ORA/eSPIA instead
- The p-values to combine come from unrelated pathway sets that cannot be paired row-wise
  into an n-by-m matrix
- Peak-level tasks like peak identification or concentration calibration are the goal
  — these are separate SMART 2.0 post-analysis modules, not pathway p-value fusion
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: Pbine
  role: Performs the p-value combination (Fisher, weighted Fisher, decorrelation)
    with numerical integration and correlation adjustment across omics p-value sequences
  repo: https://github.com/Yinchun-Lin/Pbine
- name: SMART 2.0
  role: Provides the IOPA module that generates the gene and metabolite ORA p-values
    over KEGG pathways that Pbine fuses
  repo: https://github.com/YuJenL/SMART
- name: KEGG
  role: Pathway database whose pathway set defines the common units over which gene
    and metabolite p-values are paired and combined
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:6e9a00ab19b7eeff00452c1acbaed358687393b82b493897ed87d3b28f7d2653
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yinchun-Lin/Pbine
  tools:
  - KEGG
  - Pbine
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# combined-gene-metabolite-pvalue-fusion

## Summary

A meta-analysis skill that fuses per-pathway p-values from gene-side and metabolite-side evidence (e.g., ORA results over KEGG pathways) into a single combined significance score using Pbine's Fisher or weighted Fisher p-value combination with numerical integration.

## When to use

Use when two or more omics-specific p-value sequences (e.g., gene ORA and metabolite ORA over the same KEGG pathways) must be combined per pathway into a single FDR-corrected significance call.

## When NOT to use

- Only one omics evidence stream is available, so there is nothing to combine — run a single-stream ORA/eSPIA instead
- The p-values to combine come from unrelated pathway sets that cannot be paired row-wise into an n-by-m matrix
- Peak-level tasks like peak identification or concentration calibration are the goal — these are separate SMART 2.0 post-analysis modules, not pathway p-value fusion

## Inputs

- n-by-m matrix of per-pathway p-values, one column per omics evidence source (e.g., gene ORA p-values and metabolite ORA p-values over KEGG pathways)
- optional user-specified m-by-m correlation matrix (sigma) of the combined p-values
- SMART 2.0 example breast cancer gene-expression and metabolomics data (YuJenL/SMART)

## Outputs

- Combined per-pathway test statistics and p-values from the numerical integration method
- FDR-corrected combined significance calls (e.g., 34 significant KEGG pathways at pFDR < 0.05 for the breast cancer data)
- Ranked pathway table (pathway IDs, names, p-values, significance flags) suitable for export as smart_iopa_results.csv

## How to apply

First run omics-specific pathway tests on the same pathway set (e.g., SMART 2.0 IOPA ORA for gene-expression and metabolomics data over KEGG pathways) to obtain paired p-values per pathway. Assemble these as an n-by-m p-value matrix where each row is a pathway and each column is an omics study, then call Pbine, choosing method='Int' for the numerical-integration test statistic based on Fisher's (1932) or weighted Fisher's (Good, 1955) combination; supply or estimate a correlation matrix (sigma) of p-values since gene and metabolite p-values can be correlated. If only one pathway (n=1) is tested, sigma must be provided explicitly; alternatives 'Fisher' (independence assumption) and 'Decor' (Zaykin 2002 decorrelation) are available. Finally apply FDR correction across pathways and rank results — in the SMART 2.0 breast cancer example, Pbine flagged 34 significant KEGG pathways (pFDR < 0.05), with PI3K-Akt signaling (hsa04151) most significant at 2.80e-10.

## Related tools

- **Pbine** (Performs the p-value combination (Fisher, weighted Fisher, decorrelation) with numerical integration and correlation adjustment across omics p-value sequences) — https://github.com/Yinchun-Lin/Pbine
- **SMART 2.0** (Provides the IOPA module that generates the gene and metabolite ORA p-values over KEGG pathways that Pbine fuses) — https://github.com/YuJenL/SMART
- **KEGG** (Pathway database whose pathway set defines the common units over which gene and metabolite p-values are paired and combined)

## Examples

```
library(Pbine); result <- Pbine(p = pathway_pval_matrix, method = "Int", sigma = sigma_matrix); result$pFDR <- p.adjust(result$pvalue, method = "BH")
```

## Evaluation signals

- Reproducing the SMART 2.0 breast cancer analysis yields 34 significant pathways via Pbine (vs 51 via ORA and 2 via eSPIA) at pFDR < 0.05
- The PI3K-Akt signaling pathway (hsa04151) emerges as the top hit with an FDR-corrected p-value of 2.80e-10
- For individual pathways, combined calls are consistent with component evidence, e.g., pentose phosphate pathway (hsa00030) shows significant gene ORA p = 0.02202 and metabolite ORA p = 0.02009
- When n > 1 and sigma is not supplied, Pbine successfully estimates the correlation matrix from the paired p-value sequences without error
- Output table smart_iopa_results.csv contains pathway IDs, names, p-values, and significance flags for all three methods (ORA, eSPIA, Pbine)

## Limitations

- If only one p-value pair is available (n = 1), a correlation matrix sigma must be provided by the user; it cannot be estimated
- The conventional Fisher's method assumes all p-values are independent, which is inappropriate when gene and metabolite pathway p-values are correlated
- Further biological experiments are needed to confirm the significance of novel genes and metabolites not yet documented in pathway databases

## Evidence

- [discussion] the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [discussion] the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively: "the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively"
- [readme] This matrix is composed by p-value sequences with a size of n in m studies. In each of the n rows, m p-values (i.e., p-values from m studies) will be combined.: [evidence span withheld — anchor status: external_doc]
- [readme] If n = 1, sigma must be provided. If n > 1 and sigma is not specified by a user, then sigma will be estimated based on the pairs of p-value sequence: [evidence span withheld — anchor status: external_doc]
- [intro] This knowledge gap has driven us to develop a novel multiomics pathway analysis approach that combines diverse biological data with established pathway databases: "This knowledge gap has driven us to develop a novel multiomics pathway analysis approach that combines diverse biological data with established pathway databases"
