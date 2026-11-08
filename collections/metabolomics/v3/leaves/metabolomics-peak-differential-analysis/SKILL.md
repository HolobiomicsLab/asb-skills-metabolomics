---
name: metabolomics-peak-differential-analysis
description: Use when comparing untargeted LC-MS metabolomics peak matrices between
  two labeled groups (e.g., PTB vs healthy controls) and you need per-feature log2
  fold-changes, FDR-adjusted p-values, metabolite annotation, and pathway enrichment.
when_to_use_negative:
- BiomiX does not support instrument normalization for metabolomics, so avoid relying
  on it when raw instrument-drift correction is required (QC samples can only be previewed
  in PCA/UMAP before removal)
- Proteomics or genomics data — BiomiX's analyzers cover transcriptomics, metabolomics,
  and methylomics only
- Data already reduced to an annotated, statistically tested feature table with no
  per-peak intensities to re-analyze
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: BiomiX
  role: R-based GUI tool performing the metabolomics differential peak analysis (Wilcoxon
    test, FDR correction, QC/imputation interfaces)
  repo: https://github.com/IxI-97/BiomiX
- name: TidyMass
  role: Handles fragmentation spectra for LC-MS untargeted metabolomics annotation
- name: CEU Mass Mediator
  role: Database used to annotate significant peaks by mass-to-charge ratio
- name: EnrichR
  role: Pathway/biological process enrichment of the significant metabolite list
- name: GSEA
  role: Second enrichment method to confirm enriched pathways among significant features
- name: R
  role: Underlying language/environment in which BiomiX runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:0bb698ada509421ca85830cef3d677158e777ac8e936dbc031c6075ac370fd50
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - CEU Mass Mediator
  - EnrichR
  - GSEA
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# metabolomics-peak-differential-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Single-omics metabolomics differential analysis as implemented in BiomiX: impute missing peak values, quantify peak differences between two conditions via the Wilcoxon rank-sum test with False Discovery Rate correction, annotate significant peaks by m/z against the CEU Mass Mediator database, and explore enriched biological processes with EnrichR and GSEA.

## When to use

Use when comparing untargeted LC-MS metabolomics peak matrices between two labeled groups (e.g., PTB vs healthy controls) and you need per-feature log2 fold-changes, FDR-adjusted p-values, metabolite annotation, and pathway enrichment.

## When NOT to use

- BiomiX does not support instrument normalization for metabolomics, so avoid relying on it when raw instrument-drift correction is required (QC samples can only be previewed in PCA/UMAP before removal)
- Proteomics or genomics data — BiomiX's analyzers cover transcriptomics, metabolomics, and methylomics only
- Data already reduced to an annotated, statistically tested feature table with no per-peak intensities to re-analyze

## Inputs

- Metabolomics peak intensity matrix (features x samples) with condition-labeled sample metadata (e.g., PTB vs HC)
- Fragmentation spectra files (.mgf or .mzML) for LC-MS untargeted annotation
- Optional QC-sample labels in the CONDITION column and loading-order columns in the metadata

## Outputs

- Per-peak results table with log2FC and FDR-adjusted p-value columns for every input peak
- List of significant metabolites annotated by m/z via CEU Mass Mediator
- EnrichR/GSEA pathway enrichment results for the significant feature set

## How to apply

Load the metabolomics peak matrix and sample metadata with condition labels, then run BiomiX's preview-QC interface to inspect PCA/UMAP, apply transformations, and remove outliers based on distances from PCA centroids and highly variable features before statistics. Impute missing peak values with one of BiomiX's supported methods (NIPALS, random forest, lasso, or mean/median); BiomiX 2.4+ can also handle unimputed data since MOFA tolerates missingness. For each metabolic feature, compute log2 of the ratio of median peak intensities between groups, test group differences with the Wilcoxon (Mann-Whitney) rank-sum test, and correct p-values using the False Discovery Rate method. Filter significant peaks with |log2FC| > 0.5 and adjusted p < 0.1, annotate them by m/z via the CEU Mass Mediator database (and fragmentation spectra via TidyMass, supporting .mgf files lighter than .mzML), then submit the significant metabolite list to EnrichR and GSEA to recover enriched pathways such as sphingolipid signaling and retrograde endocannabinoid signaling.

## Related tools

- **BiomiX** (R-based GUI tool performing the metabolomics differential peak analysis (Wilcoxon test, FDR correction, QC/imputation interfaces)) — https://github.com/IxI-97/BiomiX
- **TidyMass** (Handles fragmentation spectra for LC-MS untargeted metabolomics annotation)
- **CEU Mass Mediator** (Database used to annotate significant peaks by mass-to-charge ratio)
- **EnrichR** (Pathway/biological process enrichment of the significant metabolite list)
- **GSEA** (Second enrichment method to confirm enriched pathways among significant features)
- **R** (Underlying language/environment in which BiomiX runs)

## Examples

```
Rscript -e "BiomiX::runMetabolomics(peaks.csv, metadata.csv, test='wilcoxon', fdr='BH', log2FC=0.5, padj=0.1)"
```

## Evaluation signals

- Output table contains log2FC and FDR-adjusted p-value columns for every input peak, with adjusted p-values bounded in [0,1]
- Significant set is defined by |log2FC| > 0.5 and adjusted p < 0.1
- Reported pathway terms (sphingolipid signaling, retrograde endocannabinoid signaling, purine/linoleic/glycerophospholipid metabolism, phenylalanine metabolism, Fc gamma R-mediated phagocytosis) are recovered among enrichments of the significant feature set
- Expected biological direction reproduced: reductions in acylcarnitine, PC, LysoPE and TG in the PTB plasma case study
- QC samples labeled 'QC' in the CONDITION column appear in preview PCA/UMAP and are removed automatically before statistics

## Limitations

- BiomiX does not support instrument normalization for metabolomics; QC can only be previewed by labeling 'QC' in the CONDITION column before automatic removal
- Users are strongly recommended to follow guidelines and carefully review the dataset before use; no automated validation
- Users must carefully review datasets; BiomiX is described as a first attempt, not a definitive solution

## Evidence
