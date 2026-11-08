---
name: integrative-omics-pathway-analysis
description: Use when you have matched gene-expression and metabolomics data from
  a phenotype study and need to identify significantly perturbed pathways (pFDR <
  0.05) using over-representation, topology, and combined p-value evidence.
when_to_use_negative:
- You have only metabolomics data without a matched gene-expression layer — the integrative
  combination (Pbine/eSPIA) requires both omics inputs
- You need peak annotation or calibration rather than pathway-level interpretation
  — SMART 2.0's postanalysis modules (peak identification, concentration calibration)
  are separate
- Your pathways of interest are not represented in KEGG (e.g., compounds like heroin
  and nimetazepam that HMDB does not encompass) — pathway coverage limits detection
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3391
tools:
- name: SMART 2.0
  role: R package providing the IOPA module that runs ORA and eSPIA over KEGG pathways
    and exposes the Pbine combination method for integrative omics pathway analysis
  repo: https://github.com/YuJenL/SMART
- name: Pbine
  role: p-value combination method used within IOPA to integrate gene and metabolite
    pathway evidence using Fisher's / weighted Fisher's test statistics via numerical
    integration
  repo: Yinchun-Lin/Pbine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:b0981e7b9e71644a7ff327368c07552414aac416d0f806651d6bbde54b4e1305
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - Pbine
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# integrative-omics-pathway-analysis

## Summary

Run SMART 2.0's IOPA module to detect significantly enriched KEGG pathways by combining gene-expression and metabolomics evidence via ORA, eSPIA (pathway topology), and Pbine (p-value combination). It quantifies multiomics pathway perturbation, e.g. PI3K-Akt signaling (hsa04151) at FDR-corrected p = 2.80e-10 in breast cancer data.

## When to use

Use when you have matched gene-expression and metabolomics data from a phenotype study and need to identify significantly perturbed pathways (pFDR < 0.05) using over-representation, topology, and combined p-value evidence.

## When NOT to use

- You have only metabolomics data without a matched gene-expression layer — the integrative combination (Pbine/eSPIA) requires both omics inputs
- You need peak annotation or calibration rather than pathway-level interpretation — SMART 2.0's postanalysis modules (peak identification, concentration calibration) are separate
- Your pathways of interest are not represented in KEGG (e.g., compounds like heroin and nimetazepam that HMDB does not encompass) — pathway coverage limits detection

## Inputs

- Gene-expression dataset (e.g., breast cancer multiomics example data from YuJenL/SMART)
- Metabolomics dataset matched to the same phenotype samples
- KEGG pathway definitions (pathway IDs such as hsa04151 PI3K-Akt, hsa00030 pentose phosphate)
- For Pbine: an n-by-m matrix of p-values and optionally an m-by-m correlation matrix (sigma) of p-values

## Outputs

- Significant pathway lists from ORA (51 pathways in the breast cancer example, pFDR < 0.05)
- Significant pathways from eSPIA topology analysis (2 pathways in the breast cancer example)
- Combined-evidence significant pathways from Pbine (34 pathways; hsa04151 at FDR p = 2.80e-10)
- Combined results table with pathway IDs, names, p-values, and significance flags (smart_iopa_results.csv)

## How to apply

Load the gene-expression and metabolomics data (e.g., the SMART 2.0 breast cancer example dataset from the YuJenL/SMART repository) and run the IOPA module against KEGG pathways. First run over-representation analysis (ORA) separately on genes and metabolites — in the breast cancer example, gene and metabolite ORA for the pentose phosphate pathway gave p-values of 0.02202 and 0.02009. Then run eSPIA, which exploits KEGG pathway topology to detect perturbed pathways. Finally run Pbine (Yinchun-Lin/Pbine), which combines p-values via Fisher's or weighted Fisher's method computed by numerical integration, accounting for p-value correlation through a correlation matrix (sigma) estimated from paired p-value sequences or user-supplied. Compile pathway IDs, names, p-values, and significance flags (pFDR < 0.05) into a single results table (e.g., smart_iopa_results.csv).

## Related tools

- **SMART 2.0** (R package providing the IOPA module that runs ORA and eSPIA over KEGG pathways and exposes the Pbine combination method for integrative omics pathway analysis) — https://github.com/YuJenL/SMART
- **Pbine** (p-value combination method used within IOPA to integrate gene and metabolite pathway evidence using Fisher's / weighted Fisher's test statistics via numerical integration) — Yinchun-Lin/Pbine

## Evaluation signals

- ORA on the breast cancer example should report 51 significant KEGG pathways at pFDR < 0.05; eSPIA 2; Pbine 34
- The PI3K-Akt signaling pathway (hsa04151) should emerge as the most significant Pbine result with FDR-corrected p-value of 2.80e-10
- Pbine combined p-values must be more significant (or consistent with) the individual gene and metabolite ORA p-values it combines (e.g., pentose phosphate pathway p = 0.02202 and 0.02009)
- Output table must contain pathway IDs, names, p-values, and significance flags for all three methods and be saved as smart_iopa_results.csv
- If n = 1 p-value row is passed to Pbine, a user-specified sigma correlation matrix must be supplied or the call is invalid

## Limitations

- Pathway detection is bounded by KEGG/HMDB coverage: SMART 2.0 could not detect heroin and nimetazepam because HMDB does not encompass them
- Novel significant genes/metabolites not yet documented require further biological experiments to confirm
- Pbine's correlation-aware methods ('Int', 'Decor') require an estimated or user-supplied correlation matrix of p-values; only plain 'Fisher' assumes independence
- AI methodologies are not yet integrated into SMART (planned future work)

## Evidence

- [abstract] IOPA on KEGG pathways with SMART 2.0: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [discussion] PI3K-Akt as top Pbine hit: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [intro] Pathway databases supported: "combines diverse biological data with established pathway databases, including KEGG, Reactome, and WikiPathways"
- [readme] IOPA module in SMART's statistical analysis module: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
- [readme] Pbine p-value combination method: [evidence span withheld — anchor status: external_doc]
