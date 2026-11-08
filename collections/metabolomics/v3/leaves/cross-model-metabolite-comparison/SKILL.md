---
name: cross-model-metabolite-comparison
description: Use when metabolomics data exist for two or more systems modeling the
  same disease driver question — e.
when_to_use_negative:
- Only one model system is available — the cross-model concordance logic (NES agreement
  across 3 datasets) is undefined with a single dataset.
- The question is whether driver status correlates with grade/stage rather than which
  metabolic pathways differ — this skill outputs pathway enrichment, not clinical
  association tests (the paper itself found AKT1/MYC status independent of Gleason
  grade and staging).
- Untargeted features have not been mapped to KEGG-annotated metabolites — single-sample
  GSEA over KEGG metabolite sets requires metabolite-to-pathway assignments first.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: KEGG
  role: Source of the >50 metabolite pathway sets (KEGG annotation, Dataset S1) over
    which single-sample GSEA is run in each dataset before cross-model NES comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:bfec0551c669f5bbd5cbb89daabfc1d71ef164f651f00fe45510b24f800749b6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KEGG
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-model-metabolite-comparison

## Summary

Harmonizes mass-spectrometry metabolite profiles across multiple experimental models (engineered human prostate epithelial cells, transgenic mice, and human tumors) and tests shared KEGG-annotated pathway sets with single-sample GSEA to expose driver-specific metabolic fingerprints (AKT1 glycolysis vs MYC lipid dysregulation).

## When to use

Use when metabolomics data exist for two or more systems modeling the same disease driver question — e.g. engineered cell lines and in vivo models plus patient specimens — and you need to decide which metabolic pathways are genuinely associated with each oncogenic driver rather than with one model system.

## When NOT to use

- Only one model system is available — the cross-model concordance logic (NES agreement across 3 datasets) is undefined with a single dataset.
- The question is whether driver status correlates with grade/stage rather than which metabolic pathways differ — this skill outputs pathway enrichment, not clinical association tests (the paper itself found AKT1/MYC status independent of Gleason grade and staging).
- Untargeted features have not been mapped to KEGG-annotated metabolites — single-sample GSEA over KEGG metabolite sets requires metabolite-to-pathway assignments first.

## Inputs

- Mass spectrometry-based untargeted metabolite profiles (metabolite × sample matrix) from ≥2 model systems (e.g. RWPE-1-AKT1 / RWPE-1-MYC / RWPE-EV cells, transgenic mouse prostates, human tumor specimens)
- KEGG-annotated metabolite set definitions (Dataset S1 style, >50 sets)
- Driver-status labels per sample (phosphoAKT1 and MYC protein levels from immunoblot densitometry)

## Outputs

- Pathway × dataset table of normalized enrichment scores (NES) showing direction and magnitude of KEGG set enrichment in each model
- Per-pathway cross-model concordance calls (e.g. glycolysis enriched AKT1-consistently; lipid metabolism MYC-consistently)
- Validated list of driver-associated metabolites (absolute concentrations) with direction of change per condition

## How to apply

Collect the untargeted mass-spectrometry metabolite profiles for each model — here RWPE-1 cells engineered with myristoylated AKT1 or MYC, MPAKT and Lo-MYC transgenic mouse prostates, and phosphoAKT1/MYC-immunoblotted human T2 tumors — plus KEGG-annotated metabolite sets (over 50 sets as Dataset S1). Run single-sample Gene Set Enrichment Analysis on each dataset separately, using gene set-size-normalized enrichment scores (NES) to capture both the extent and the direction of enrichment per pathway. Compare NES values across all three datasets side by side to find pathways that consistently enrich under one driver (e.g. glycolysis and pentose phosphate under AKT1; lipid metabolism under MYC) and record where the models disagree. Validate selected differentially accumulated metabolites with absolute quantitation assays before committing to the cross-model conclusion. Optionally cluster the driver-status groups (phosphoAKT1-high/MYC-low, phosphoAKT1-low/MYC-high, etc.) and confirm segregation with a Fisher test (p<0.01) to ensure the contrast being compared is real.

## Related tools

- **KEGG** (Source of the >50 metabolite pathway sets (KEGG annotation, Dataset S1) over which single-sample GSEA is run in each dataset before cross-model NES comparison)

## Evaluation signals

- Each dataset (cells, mice, human tumors) yields its own NES value per KEGG metabolite set, and the glycolysis set shows enrichment in the AKT1-activated condition while lipid-metabolism sets show enrichment in the MYC-overexpressing condition in ≥2 datasets.
- Selected driver-associated metabolites survive absolute quantitation validation, not just untargeted MS detection.
- Clustering of phosphoAKT1/MYC subgroups is confirmed statistically (Fisher test p<0.01) before enrichment contrasts are interpreted.
- NES values are gene set-size-normalized, so cross-pathway and cross-dataset comparisons are not biased by pathway size.
- Driver-status calls are anchored to measured phosphoAKT1 and MYC immunoblot densitometry, not inferred labels.

## Limitations

- Untargeted metabolomics must be validated with absolute quantitation for specific metabolites before cross-model claims are treated as quantitative.
- The paper found AKT1/MYC status independent of Gleason grade and pathologic staging, so concordant metabolic fingerprints should not be over-interpreted as grade- or stage-linked.
- Pathway-level conclusions depend on the KEGG annotation coverage of detected features; metabolites without KEGG assignment are excluded from the GSEA.

## Evidence

- [abstract] Integrative analysis of these metabolomic datasets revealed: "Mass spectrometry-based metabolite profiling was performed on immortalized human prostate epithelial cells transformed by AKT1 or MYC, transgenic mice"
- [results] single-sample GSEA over >50 KEGG sets: "Over 50 metabolite sets (KEGG annotation - Dataset S1) were tested using single-sample Gene Set Enrichment Analysis (GSEA)"
- [results] NES across 3 datasets: "Gene set-size-normalized enrichment scores (NES) from GSEA were used to determine the extent and direction of enrichment for the 3 data sets"
- [abstract] absolute quantitation validation: "were validated using absolute quantitation assays"
- [results] driver-status segregation confirmed statistically: "Human tumors fell in 3 clusters (defined by Silhouette analysis), where the phosphoAKT1-low/MYC-high tumors and the phosphoAKT1-high/MYC-low tumors differentially segregated (Fisher test, p<0.01)"
