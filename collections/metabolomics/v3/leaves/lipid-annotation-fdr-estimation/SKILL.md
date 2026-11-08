---
name: lipid-annotation-fdr-estimation
description: Use when you have MS/MS feature tables from LC-MS lipidomics (e.g., RBL-2H3,
  mixture, human sera, zebrafish) and need to quantify annotation confidence via an
  estimated FDR against the LipidIN theoretical library.
when_to_use_negative:
- Your goal is downstream quantification, differential analysis, or pathway enrichment
  rather than annotation confidence estimation
- Your data requires Paternò–Büchi double-bond localization rather than standard chain-composition
  annotation scoring
- Input is already a finalized annotation table without the raw MS/MS features needed
  to re-score EQ+LCI hits at the 2.4 cutoff
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Framework providing the EQ querying module, LCI RT-rule model, and WMYn fingerprint
    regeneration used to annotate features and estimate FDR
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Alternative annotation workflow; EQ combined with MS-DIAL public database
    was benchmarked against the MS-DIAL program itself
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:8c07edc4f794bd7c8bac5838528af483f7c071a61d4826f57064913e9f142c96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - LipidIN EQ module
  - LipidIN LCI module
  - LipidIN WMYn module
  - MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# lipid-annotation-fdr-estimation

## Summary

Estimate the false discovery rate of LipidIN EQ+LCI lipid annotations on LC-MS/MS feature tables by combining MS2 library search, LCI relative-RT-rule filtering, and strict ECN-rule/feature-peak criteria; the reference benchmark achieves ~5.7% average FDR across 8,923 annotated lipids in four datasets.

## When to use

Use when you have MS/MS feature tables from LC-MS lipidomics (e.g., RBL-2H3, mixture, human sera, zebrafish) and need to quantify annotation confidence via an estimated FDR against the LipidIN theoretical library.

## When NOT to use

- Your goal is downstream quantification, differential analysis, or pathway enrichment rather than annotation confidence estimation
- Your data requires Paternò–Büchi double-bond localization rather than standard chain-composition annotation scoring
- Input is already a finalized annotation table without the raw MS/MS features needed to re-score EQ+LCI hits at the 2.4 cutoff

## Inputs

- Raw MS/MS feature tables from RBL-2H3 cells, mixture dataset, human sera, and zebrafish tissues
- LipidIN general theoretical spectral library (pos_ALL.rda / neg_ALL.rda, or pos_common.pkl / neg_common.pkl)
- Optional LipidIN PB spectral library (PB_PX_H.pkl, common_PX_CH3COO.pkl + MS1 index)

## Outputs

- Per-dataset annotation tables with lipid subclass predictions
- Aggregated average FDR summary (~5.69% under strict criteria)
- annotation_report.csv matching Fig. 3e-h and Supplementary Data 6-7

## How to apply

Load raw MS/MS feature tables and download the LipidIN spectral libraries from Zenodo (pos_ALL.rda/neg_ALL.rda for R, or pos_common.pkl/neg_common.pkl for Python, ~24.6 million spectra). Run the EQ (expeditious querying) module — which uses hash tables and bisection methods for linear-time search against chain compositions + C=C double-bond positions — then apply the LCI module's three relative retention time rules with a best-first search heuristic to predict subclass and remove false positives. Enforce the strict ECN-rule (equivalent carbon number) and feature-peak criteria at a cutoff threshold of 2.4 to filter inconsistent annotations. Optionally use the WMYn-regenerated 5th-level fingerprint library to boost recall and confirm low-confidence lipids. Compute per-dataset estimated FDR, aggregate across datasets, and save annotation_report.csv; correctness is judged by reproducing the reported 5.69% average FDR and 471 unique mixture-dataset lipids.

## Related tools

- **LipidIN** (Framework providing the EQ querying module, LCI RT-rule model, and WMYn fingerprint regeneration used to annotate features and estimate FDR) — https://github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Alternative annotation workflow; EQ combined with MS-DIAL public database was benchmarked against the MS-DIAL program itself)

## Examples

```
wget -O lipidin_spectral_libs.zip https://zenodo.org/records/21421866
```

## Evaluation signals

- Average estimated FDR across the four datasets should be ~5.69% at the 2.4 cutoff threshold under strict criteria
- Total annotated lipids should reach ~8,923 across species; 471 unique lipids expected in the mixture dataset
- ECN-rule and feature-peak filtering must remove inconsistent annotations before FDR computation
- annotation_report.csv must contain both per-dataset and aggregated annotation/FDR results matching Supplementary Data 6-7
- Query throughput sanity check: ~70 billion spectral comparisons complete in under 1 second

## Limitations

- The LCI model relies on RT rules, so results depend on chromatographic reproducibility across sample matrices and instruments
- Similarity-based annotations (MS-DIAL, LipidSearch, MS entropy) can cause feature redundancy compared to LipidIN EQ
- Estimated FDR (~5.7%) is a model-estimated value, not a decoy-database empirical FDR

## Evidence

- [abstract] developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate, covering 8923 lipids cross: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate, covering 8923 lipids cross"
- [readme] Annotation false discovery rate | 5.7 % (estimated), across 8,923 lipids: "Annotation false discovery rate | 5.7 % (estimated), across 8,923 lipids"
- [discussion] regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
- [methods] Download the spectral libraries from **[Zenodo → https://zenodo.org/records/21421866](https://zenodo.org/records/21421866)**: "Download the spectral libraries from **[Zenodo → https://zenodo.org/records/21421866](https://zenodo.org/records/21421866)**"
