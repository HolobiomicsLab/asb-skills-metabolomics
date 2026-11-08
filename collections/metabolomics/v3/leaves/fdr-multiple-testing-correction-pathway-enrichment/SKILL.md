---
name: fdr-multiple-testing-correction-pathway-enrichment
description: Use when per-pathway raw p-values (e.g., from eSPIA impact analysis or
  gene/metabolite ORA) must be corrected across many pathways, or when p-values from
  joint gene-metabolite tests need combination before calling significance.
when_to_use_negative:
- When only a single hypothesis or a handful of pathways is tested and no multiple-testing
  correction is needed
- When p-values are not paired across data types/studies and no correlation structure
  can be estimated, making Pbine's 'Int'/'Decor' methods inappropriate (plain Fisher's
  method assumes independence)
- When the goal is raw effect-size ranking of pathways rather than corrected statistical
  significance calls
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2269
tools:
- name: SMART 2.0
  role: R tool whose integrative omics pathway analysis (IOPA/eSPIA) generates per-pathway
    p-values that are then FDR-corrected and compared against ORA calls
  repo: https://github.com/YuJenL/SMART
- name: Pbine
  role: R tool for combining p-values from multiple studies/tests (Fisher's, weighted
    Fisher's, decorrelation, or numerical integration with a correlation matrix sigma)
  repo: https://github.com/Yinchun-Lin/Pbine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:493e5d44f509cbccc518f0b688445882ce5eac013f7893b97dca245cd0b5aedf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - KEGGREST/KEGGgraph
  - Pbine
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# fdr-multiple-testing-correction-pathway-enrichment

## Summary

Applies multiple-testing correction (FDR/Bonferroni) and p-value combination (Fisher's method, Pbine numerical integration) to pathway analysis p-values, as used in SMART 2.0's integrative omics pathway analysis (IOPA) to call significant gene-and-metabolite KEGG pathways.

## When to use

Use when per-pathway raw p-values (e.g., from eSPIA impact analysis or gene/metabolite ORA) must be corrected across many pathways, or when p-values from joint gene-metabolite tests need combination before calling significance.

## When NOT to use

- When only a single hypothesis or a handful of pathways is tested and no multiple-testing correction is needed
- When p-values are not paired across data types/studies and no correlation structure can be estimated, making Pbine's 'Int'/'Decor' methods inappropriate (plain Fisher's method assumes independence)
- When the goal is raw effect-size ranking of pathways rather than corrected statistical significance calls

## Inputs

- Vector or n-by-m matrix of raw per-pathway p-values (e.g., gene and metabolite ORA p-values; eSPIA perturbation p-values)
- Optional correlation matrix (sigma) of p-values for Pbine's 'Int' or 'Decor' methods

## Outputs

- Per-pathway FDR-corrected (and/or Bonferroni-corrected) p-values with significant pathways flagged at pFDR < 0.05 (e.g., written to espia_results.csv)
- Combined gene-metabolite p-values per pathway (Fisher or Pbine test statistic)

## How to apply

Collect raw per-pathway p-values from both the perturbation/impact analysis (eSPIA, which extends SPIA to joint gene-and-metabolite KEGG pathway topology) and the over-representation analysis (ORA) on the same gene and metabolite feature sets. Combine gene and metabolite p-values per pathway using Fisher's method, or the Pbine numerical-integration method (which can account for correlated p-values via an estimated or user-supplied correlation matrix sigma, and also supports the Zaykin decorrelation approach). Then apply FDR (or Bonferroni) correction across all pathways and select significant pathways at pFDR < 0.05. In the SMART 2.0 breast cancer IOPA example, this pipeline yielded 2 significant eSPIA pathways versus 51 significant ORA pathways, illustrating that correction over a large ORA set and topology-aware testing produce very different calls. Interpret combined significance per pathway, e.g., hsa04151 (PI3K-Akt signaling pathway) reached an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method.

## Related tools

- **SMART 2.0** (R tool whose integrative omics pathway analysis (IOPA/eSPIA) generates per-pathway p-values that are then FDR-corrected and compared against ORA calls) — https://github.com/YuJenL/SMART
- **Pbine** (R tool for combining p-values from multiple studies/tests (Fisher's, weighted Fisher's, decorrelation, or numerical integration with a correlation matrix sigma)) — https://github.com/Yinchun-Lin/Pbine

## Examples

```
Pbine(p = pvals_matrix, method = "Int", sigma = sigma)  # or method = "Fisher"; then FDR-correct across pathways and select pFDR < 0.05
```

## Evaluation signals

- On the SMART 2.0 breast cancer data, the corrected eSPIA analysis should identify 2 significant pathways at pFDR < 0.05, while ORA on the same sets identifies 51 significant pathways at pFDR < 0.05
- The pentose phosphate pathway's separate gene and metabolite ORA p-values should reproduce as 0.02202 and 0.02009, respectively
- The PI3K-Akt signaling pathway (hsa04151) should emerge as the most significant with an FDR-corrected p-value of 2.80 × 10–10 under the Pbine method
- Output table (e.g., espia_results.csv) must contain per-pathway ID, impact statistic, raw p-value, FDR-corrected p-value, and ORA comparison flag
- Pbine output should match the chosen method ('Int', 'Fisher', or 'Decor') and, for n > 1 without a user-supplied sigma, the estimated correlation matrix should be based on pairs of the observed p-value sequences

## Limitations

- Fisher's method assumes all p-values are independent; correlated p-values require Pbine's numerical-integration method with an estimated or user-supplied correlation matrix
- Bonferroni correction over a large pathway set (e.g., ORA detecting 51 significant pathways at pFDR < 0.05) is highly sensitive to the number of tests and contrasts sharply with topology-aware impact analysis
- For Pbine, if n = 1 the correlation matrix sigma must be provided by the user
- Corrected significance calls are statistical, not biological proof; the article notes further biological experiments are warranted to confirm the significance of undocumented genes and metabolites

## Evidence

- [discussion] the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [readme] In our method, p-values are combined based on the test statistic: Fisher's p-value combination (Fisher, 1932) or weighted Fisher's p-value combination (Good, 1955).: [evidence span withheld — anchor status: external_doc]
- [readme] Three methods of p-value combination are provided. "Int": the proposed numerical integration method; "Fisher": Fisher's method that it assumes all p-values are independent: [evidence span withheld — anchor status: external_doc]
- [discussion] the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively: "the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively"
