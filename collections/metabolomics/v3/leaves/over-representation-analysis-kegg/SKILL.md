---
name: over-representation-analysis-kegg
description: Use when you have a precomputed list of significant gene-expression or
  metabolite identifiers (e.g., KEGG IDs) and want to test whether KEGG pathways are
  over-represented, optionally feeding p-values into eSPIA or Pbine.
when_to_use_negative:
- Input is raw spectral data (e.g., .raw, .d, mzXML) rather than a list of significant
  gene/metabolite identifiers — preprocessing and peak analysis modules should run
  first.
- You need topology-aware pathway impact scores rather than simple over-representation
  counts — eSPIA over KEGG pathway topology is the appropriate follow-up.
- Pathway database of interest is Reactome or WikiPathways rather than KEGG — ORA
  mapping here is specific to KEGG pathways.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0605
tools:
- name: SMART 2.0
  role: R tool providing the IOPA module that runs ORA against KEGG pathways and integrates
    results with eSPIA and Pbine
  repo: https://github.com/YuJenL/SMART
- name: Pbine
  role: Meta-analysis method combining the gene and metabolite ORA p-values per pathway
    via Fisher's / weighted Fisher's p-value combination with numerical integration
  repo: https://github.com/Yinchun-Lin/Pbine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:11fbda40b2864f91cc4681b7d7ce632141d2fc145b2a99b532cd79152bd4f1fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - KEGG
  - Pbine
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# over-representation-analysis-kegg

## Summary

Over-representation analysis (ORA) of significantly altered genes/metabolites against KEGG pathways, as implemented in the SMART 2.0 IOPA module, to identify enriched pathways (e.g., 51 significant pathways in the breast cancer multiomics demonstration, pFDR < 0.05).

## When to use

Use when you have a precomputed list of significant gene-expression or metabolite identifiers (e.g., KEGG IDs) and want to test whether KEGG pathways are over-represented, optionally feeding p-values into eSPIA or Pbine.

## When NOT to use

- Input is raw spectral data (e.g., .raw, .d, mzXML) rather than a list of significant gene/metabolite identifiers — preprocessing and peak analysis modules should run first.
- You need topology-aware pathway impact scores rather than simple over-representation counts — eSPIA over KEGG pathway topology is the appropriate follow-up.
- Pathway database of interest is Reactome or WikiPathways rather than KEGG — ORA mapping here is specific to KEGG pathways.

## Inputs

- Gene-expression differential-abundance results (gene identifiers/significance)
- Metabolomics differential-abundance results (metabolite KEGG IDs, e.g., C02630 for 2-hydroxyglutarate)
- KEGG pathway database (pathway ID to gene/metabolite mapping)

## Outputs

- Per-pathway ORA p-values and FDR-corrected pFDR values with significance flags (pFDR < 0.05)
- Combined results table (e.g., smart_iopa_results.csv) with pathway IDs, names, p-values, and significance flags across ORA, eSPIA, and Pbine

## How to apply

Load the gene-expression and metabolomics data (e.g., the SMART 2.0 breast cancer example dataset from YuJenL/SMART) into the SMART 2.0 IOPA module and run ORA against KEGG pathways to obtain per-pathway p-values and significance flags (FDR-corrected pFDR < 0.05; 51 significant pathways were detected in the breast cancer demonstration). Pathway IDs, names, p-values, and significance flags are compiled into a results table (smart_iopa_results.csv). ORA output for the same pathway set is then used downstream: eSPIA incorporates KEGG pathway topology (2 significant pathways in the demonstration), and Pbine combines the gene and metabolite ORA p-values via Fisher/weighted-Fisher p-value combination with numerical integration (34 significant pathways; the PI3K-Akt signaling pathway, hsa04151, was most significant at FDR-corrected p = 2.80e-10). Correctness is judged by whether significant pathway counts, pFDR values, and top hits (e.g., hsa04151) are reproducible against the reported values.

## Related tools

- **SMART 2.0** (R tool providing the IOPA module that runs ORA against KEGG pathways and integrates results with eSPIA and Pbine) — https://github.com/YuJenL/SMART
- **Pbine** (Meta-analysis method combining the gene and metabolite ORA p-values per pathway via Fisher's / weighted Fisher's p-value combination with numerical integration) — https://github.com/Yinchun-Lin/Pbine

## Evaluation signals

- On the SMART 2.0 breast cancer example dataset, ORA should report 51 significant pathways at pFDR < 0.05.
- Pbine on the gene+metabolite ORA p-values should report 34 significant pathways, with PI3K-Akt signaling pathway (hsa04151) most significant at FDR-corrected p = 2.80e-10.
- eSPIA over KEGG pathway topology should report 2 significant pathways.
- The combined results file (smart_iopa_results.csv) must preserve pathway IDs, names, p-values, and significance flags for all three methods, including the hsa04151 record.
- Cross-check individual gene/metabolite ORA p-values, e.g., pentose phosphate pathway (hsa00030) gene and metabolite tests at p = 0.02202 and p = 0.02009.

## Limitations

- Significance of undocumented genes/metabolites requires further biological experimental confirmation.
- Metabolite identifiers must map to KEGG; compounds absent from reference databases (e.g., heroin and nimetazepam not in HMDB) cannot be annotated or included in the analysis.
- Pbine requires a correlation matrix of p-values (or sufficient paired p-value sequences to estimate one); with n = 1, sigma must be supplied by the user.

## Evidence

- [results] Applying IOPA to the breast cancer multiomics data, SMART 2.0 detected 51 significant pathways via ORA, two via eSPIA, and 34 via Pbine (all pFDR < 0.05).: [evidence span withheld — anchor status: not_found]
- [discussion] the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [readme] Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA).: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
- [abstract] The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART.: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [discussion] the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively: "the gene and metabolite ORA tests for the pentose phosphate pathway exhibit significance, with p-values of 0.02202 and 0.02009, respectively"
