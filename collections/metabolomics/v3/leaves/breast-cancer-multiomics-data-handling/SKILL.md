---
name: breast-cancer-multiomics-data-handling
description: Use when handling paired gene-expression and metabolomics data (e.g.,
  the SMART 2.0 breast cancer example) to run integrative KEGG pathway analysis via
  ORA, eSPIA, and Pbine and compare significance counts.
when_to_use_negative:
- Input is narcotics/untargeted peak data where the goal is peak identification or
  one-to-one drug identification (SMART 2.0 cannot detect heroin and nimetazepam because
  HMDB does not include them), not pathway enrichment
- Data are targeted metabolomics calibration/concentration-calibration experiments
  rather than paired gene-expression and metabolomics profiles suitable for IOPA
- Analysis targets Reactome or WikiPathways instead of KEGG pathways; this skill's
  reproduction targets are defined over KEGG pathway IDs (hsa*)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3518
tools:
- name: SMART 2.0
  role: R tool providing the IOPA module (ORA, eSPIA, Pbine) within its Statistical
    Analysis module; also supplies the breast cancer example data
  repo: https://github.com/YuJenL/SMART
- name: Pbine
  role: Meta-analysis tool combining gene and metabolite ORA p-values via Fisher's
    or weighted Fisher's combination with numerical integration and a p-value correlation
    matrix (sigma)
  repo: https://github.com/Yinchun-Lin/Pbine
- name: KEGG
  role: Pathway database over which ORA, eSPIA, and Pbine significance are computed
    (e.g., hsa04151 PI3K-Akt, hsa00030 pentose phosphate pathway)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:49efc016e395bbb94223f111bd1c03a24d986a8487e808d6c80a790b4fc80b8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - KEGG
  - Pbine
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# breast-cancer-multiomics-data-handling

## Summary

Reproduces the SMART 2.0 integrative omics pathway analysis (IOPA: ORA, eSPIA, and Pbine over KEGG pathways) on the breast cancer gene-expression and metabolomics example dataset from the YuJenL/SMART repository, yielding a combined significance table.

## When to use

Use when handling paired gene-expression and metabolomics data (e.g., the SMART 2.0 breast cancer example) to run integrative KEGG pathway analysis via ORA, eSPIA, and Pbine and compare significance counts.

## When NOT to use

- Input is narcotics/untargeted peak data where the goal is peak identification or one-to-one drug identification (SMART 2.0 cannot detect heroin and nimetazepam because HMDB does not include them), not pathway enrichment
- Data are targeted metabolomics calibration/concentration-calibration experiments rather than paired gene-expression and metabolomics profiles suitable for IOPA
- Analysis targets Reactome or WikiPathways instead of KEGG pathways; this skill's reproduction targets are defined over KEGG pathway IDs (hsa*)

## Inputs

- Breast cancer gene-expression data matrix (SMART 2.0 example dataset from YuJenL/SMART)
- Breast cancer metabolomics data matrix (SMART 2.0 example dataset from YuJenL/SMART)
- KEGG pathway database annotations (including hsa04151 PI3K-Akt signaling pathway)

## Outputs

- ORA pathway results table (51 significant KEGG pathways, pFDR < 0.05)
- eSPIA pathway results table (2 significant KEGG pathways, pFDR < 0.05)
- Pbine integrated gene-metabolite pathway results table (34 significant KEGG pathways, pFDR < 0.05)
- smart_iopa_results.csv combining pathway IDs, names, p-values, and significance flags

## How to apply

Load the breast cancer gene-expression and metabolomics data distributed as example data in the YuJenL/SMART R repository, and run the SMART 2.0 IOPA module (in the Statistical Analysis module, which also performs ANCOVA and PLS/PLS-DA). Run ORA against KEGG pathways to obtain 51 significant pathways, then run eSPIA over KEGG pathway topology to obtain 2 significant pathways, and run the Pbine method (Fisher's or weighted Fisher's p-value combination via numerical integration, with a p-value correlation matrix sigma estimated from paired p-value sequences) to integrate gene and metabolite evidence into 34 significant pathways. Apply an FDR-corrected significance threshold of pFDR < 0.05 to all three methods; verify that the PI3K-Akt signaling pathway (hsa04151) is the most significant Pbine result at FDR-corrected p = 2.80 × 10^-10. Compile pathway IDs, names, p-values, and significance flags from all three methods into smart_iopa_results.csv.

## Related tools

- **SMART 2.0** (R tool providing the IOPA module (ORA, eSPIA, Pbine) within its Statistical Analysis module; also supplies the breast cancer example data) — https://github.com/YuJenL/SMART
- **Pbine** (Meta-analysis tool combining gene and metabolite ORA p-values via Fisher's or weighted Fisher's combination with numerical integration and a p-value correlation matrix (sigma)) — https://github.com/Yinchun-Lin/Pbine
- **KEGG** (Pathway database over which ORA, eSPIA, and Pbine significance are computed (e.g., hsa04151 PI3K-Akt, hsa00030 pentose phosphate pathway))

## Examples

```
devtools::install_github("YuJenL/SMART"); library(SMART)  # then load the breast cancer example data and run the IOPA module (ORA, eSPIA, Pbine) over KEGG pathways
```

## Evaluation signals

- ORA reports exactly 51 significant KEGG pathways at pFDR < 0.05 on the breast cancer multiomics data
- eSPIA reports exactly 2 significant KEGG pathways at pFDR < 0.05
- Pbine reports exactly 34 significant KEGG pathways, with hsa04151 (PI3K-Akt signaling pathway) as the most significant at FDR-corrected p = 2.80 × 10^-10
- smart_iopa_results.csv contains rows for all three methods with pathway IDs, names, p-values, and significance flags, preserving the PI3K-Akt record
- Gene and metabolite ORA p-values for the pentose phosphate pathway (hsa00030) are 0.02202 and 0.02009, respectively, providing per-pathway spot checks

## Limitations

- Significance counts are specific to the SMART 2.0 example breast cancer dataset and KEGG; changes in input data, annotation version, or FDR threshold (0.05) will change counts
- The pentose phosphate pathway reaches only marginal significance (p = 0.02202 gene, p = 0.02009 metabolite ORA), so near-threshold pathway calls are sensitive to method choice
- Novel genes and metabolites flagged by the analysis lack documented significance; further biological experiments are warranted to confirm them

## Evidence

- [abstract] This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities.: "This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities."
- [discussion] PI3K-Akt Pbine significance: "the PI3K-Akt signaling pathway (hsa04151) stood out as the most significant, with an FDR-corrected p-value of 2.80 × 10–10 using the Pbine method"
- [readme] Pbine p-value combination method: [evidence span withheld — anchor status: external_doc]
- [readme] SMART statistical analysis module includes IOPA: "Statistical Analysis: Perform association analysis (ANCOVA), classification analysis (PLS/PLS-DA), and integrative omics pathway analysis (IOPA)."
- [readme] Pbine correlation handling: [evidence span withheld — anchor status: external_doc]
