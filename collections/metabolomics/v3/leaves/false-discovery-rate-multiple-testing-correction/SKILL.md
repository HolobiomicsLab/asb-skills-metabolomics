---
name: false-discovery-rate-multiple-testing-correction
description: Use when a spatial metabolomics or multi-omics analysis tests thousands
  of m/z peaks, metabolite-gene pairs, or pathways simultaneously and needs FDR-adjusted
  significance before ranking or reporting hits.
when_to_use_negative:
- Input is already an FDR-adjusted results table from a previous correction step (double
  correction would be applied).
- The analysis tests only one or a few metabolites/pathways with no multiple-testing
  burden, so FDR adjustment is unnecessary.
- The task is exploratory visualisation (e.g. ion image plotting) rather than statistical
  hypothesis testing that requires significance calls.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: SpaMTP
  role: Implements the downstream statistical analyses (differential peak expression,
    pathway analysis, correlation analysis) and computes false discovery rate as part
    of the per-m/z statistics returned with each result table.
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: edgeR
  role: Performs the differential testing on pseudo-bulked m/z intensities across
    spatial groups, whose output feeds the FDR correction.
- name: Seurat
  role: Provides the cell-centric object framework and clustering used to define the
    spatial groups whose pseudo-bulked values are differentially tested and FDR-corrected.
- name: R
  role: Language environment in which SpaMTP, edgeR and the FDR-adjusted statistics
    are executed.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:eddb2e89e80ed93e45ff4b636768ff6a0adb8f83bbedd0fe54870bb99530c44e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - Seurat
  - SpaMTP
  - edgeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# false-discovery-rate-multiple-testing-correction

## Summary

Applies false-discovery rate (FDR) correction to the many parallel statistical tests performed in spatial metabolomics pipelines (differential m/z peak expression, pathway enrichment, multi-omics correlations), so that reported statistics reflect the full multiple-testing burden rather than nominal per-peak values.

## When to use

Use when a spatial metabolomics or multi-omics analysis tests thousands of m/z peaks, metabolite-gene pairs, or pathways simultaneously and needs FDR-adjusted significance before ranking or reporting hits.

## When NOT to use

- Input is already an FDR-adjusted results table from a previous correction step (double correction would be applied).
- The analysis tests only one or a few metabolites/pathways with no multiple-testing burden, so FDR adjustment is unnecessary.
- The task is exploratory visualisation (e.g. ion image plotting) rather than statistical hypothesis testing that requires significance calls.

## Inputs

- Pseudo-bulked relative intensity matrix of m/z peaks per spatial cluster (e.g. from edgeR-style differential testing in SpaMTP)
- Per-feature statistics table from differential expression, pathway enrichment, or correlation analysis (p-values, fold-changes)
- Annotated m/z intensity matrix with metabolite labels (SpaMTP SpatialMetabolomics object, e.g. spotted_Annotated_H_Cl_Adducts.RDS)
- Cluster labels for tissue regions or spatial spots (urothelium, muscle, adventitial layer etc.)

## Outputs

- Ranked table of differentially expressed m/z peaks with fold-change, FDR-adjusted p-value, and annotation columns (e.g. dem_results_table.csv)
- Filtered list of significant features that survive FDR correction
- FDR-adjusted pathway enrichment results when applied to metabolite-gene pathway statistics

## How to apply

Attach FDR correction to any SpaMTP step that emits a table of per-feature statistics. For differential m/z peak expression, SpaMTP pseudo-bulks relative metabolite intensities per spatial region/cluster (e.g. urothelium, muscle, adventitial layer of the mouse urinary bladder MALDI dataset), runs edgeR differential testing on the pooled values, then computes per-m/z fold-change and false discovery rate, retaining only significantly differentially expressed peaks for the final ranked table. The same adjustment applies to pathway enrichment from metabolite-gene mappings (e.g. RaMP/HMDB annotations) and to correlation analysis across spatial-omics modalities, where the large feature count makes unadjusted p-values misleading. Judging correct application: the output table must contain an FDR column alongside fold-change, the number of reported significant peaks must be consistent with the correction applied (e.g. ~70 DEMs between the three tissue-specific bladder clusters), and ranked hits should survive the correction rather than rely on raw p-values.

## Related tools

- **SpaMTP** (Implements the downstream statistical analyses (differential peak expression, pathway analysis, correlation analysis) and computes false discovery rate as part of the per-m/z statistics returned with each result table.) — github.com/GenomicsMachineLearning/SpaMTP
- **edgeR** (Performs the differential testing on pseudo-bulked m/z intensities across spatial groups, whose output feeds the FDR correction.)
- **Seurat** (Provides the cell-centric object framework and clustering used to define the spatial groups whose pseudo-bulked values are differentially tested and FDR-corrected.)
- **R** (Language environment in which SpaMTP, edgeR and the FDR-adjusted statistics are executed.)

## Examples

```
dem <- SpaMTP::FindAllDEMs(spatial_obj, clusters = c("urothelium", "muscle", "adventitial"), n_pools = 10, seed = 42); write.csv(dem, "dem_results_table.csv", row.names = FALSE)
```

## Evaluation signals

- The results table includes an explicit false discovery rate (FDR) column alongside fold-change and raw statistics, not just nominal p-values.
- The number of significant features reported is consistent with the correction applied — e.g. ~70 significantly differentially expressed m/z peaks between the three tissue-specific bladder clusters, including PC(34:1) up-regulated in urothelium and SM(34:1) up-regulated in muscle.
- Ranked hits (e.g. top DEMs, enriched pathways) survive the FDR threshold rather than being selected by raw p-value alone.
- FDR is applied consistently across every table of parallel tests the pipeline produces (differential expression, pathway, correlation outputs), not only to one section.
- Fixed pseudo-bulking parameters (number of pools, random seed) are recorded so the FDR-adjusted results are reproducible.

## Limitations

- SpaMTP's annotation and downstream statistics depend on upstream m/z annotation quality; features that fail annotation limits the interpretability of FDR-adjusted hits.
- METASPACE and pySM optimise for high-resolution instrument data (FT-ICR, orbitrap) and do not yet apply to most TOF-analyser data, which affects which SM datasets can reach the FDR-corrected statistical stage.
- No specific SpaMTP version, DOI, or release identifier is given for the vignette sections, and no quantified benchmark of the FDR behaviour itself is provided, so users must record their own versions and settings.
- Python SM packages (pySM, MALDIpy) were not designed for interconnection, limiting ability to run the full end-to-end pipeline including FDR-corrected statistics outside the R ecosystem.

## Evidence

- [methods] Differential metabolite expression via pseudo-bulk edgeR testing: [evidence span withheld — anchor status: not_found]
- [methods] FDR reported per m/z in DEM table: [evidence span withheld — anchor status: not_found]
- [intro] Unadjusted p-values insufficient without FDR: [evidence span withheld — anchor status: not_found]
- [readme] edgeR dependency via conda: [evidence span withheld — anchor status: external_doc]
- [readme] SpaMTP supports differential metabolite expression: [evidence span withheld — anchor status: external_doc]
