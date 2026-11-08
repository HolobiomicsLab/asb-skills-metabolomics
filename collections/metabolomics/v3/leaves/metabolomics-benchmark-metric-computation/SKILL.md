---
name: metabolomics-benchmark-metric-computation
description: Reference for quantifying annotation quality when a metabolomics annotator
  proposes KEGG compounds per LC-MS peak and a ground-truth compound set per assay
  exists; needed to reproduce per-assay TP/FP/FN tables, sensitivity (e.g. 0.63 vs
  xMSannotator 0.55), F1 (up to 0.49 for mWISE vs 0.
when_to_use_negative:
- Input is a raw LC-MS feature table (m/z, intensity, retention time) before any candidate
  annotation has been produced — annotation metrics require a proposed-compound list,
  not peak intensities
- The goal is to evaluate the annotator's peak-matching or clustering stage (e.g.
  cluster-based filter reducing Assay 2 FP from 17598 to 1671) rather than the final
  entity-level annotation quality after diffusion prioritization
- Only one dataset is available without a reference compound set or with no per-assay
  TP/FP/FN ground truth — the metric cannot be computed and an alternative evaluation
  (e.g. only structural closeness) is needed
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: mWISE
  role: Source of per-assay candidate annotation lists (top three KEGG compounds per
    LC-MS peak) that the metrics are computed against, using the FELLA or RClass diffusion
    configurations
  repo: https://github.com/b2slab/mWISE
- name: FELLA
  role: Provides the biological network used in the mWISE diffusion configuration
    whose raw-score annotations are one of the benchmark candidates (mWISE mean sensitivity
    0.63 SD 0.07 vs xMSannotator 0.55 SD 0.19)
- name: Diffu Stats
  role: R package used by mWISE to diffuse the input label vector in a given network,
    producing the raw/z-normalized diffusion scores that feed the prioritized candidate
    lists being benchmarked
- name: xMSannotator
  role: Baseline annotation algorithm whose per-assay TP/FP/FN counts and F1 (best
    0.22, Assay 6) and sensitivity (0.55 SD 0.19) are computed and compared against
    mWISE in the same benchmark tables
- name: R
  role: Language/runtime in which the mWISE benchmark tables and diffusion-based annotation
    are implemented (mWISE is an R package; rmarkdown used for the vignette)
  repo: https://github.com/b2slab/mWISE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:56c8e852f7ae4e301222455f35464fe492e3e797428f750e34bcefa782d5083f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  repo_url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# metabolomics-benchmark-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes entity-level sensitivity, specificity, precision, F1, and Tanimoto structural-similarity ratios for LC-MS feature annotation methods (e.g. mWISE vs xMSannotator) against reference compound sets, exactly as done in the mWISE paper's benchmark of FELLA/RClass diffusion prioritizations.

## When to use

Reference for quantifying annotation quality when a metabolomics annotator proposes KEGG compounds per LC-MS peak and a ground-truth compound set per assay exists; needed to reproduce per-assay TP/FP/FN tables, sensitivity (e.g. 0.63 vs xMSannotator 0.55), F1 (up to 0.49 for mWISE vs 0.22 for xMSannotator), and Tanimoto closeness of proposed vs original structures.

## When NOT to use

- Input is a raw LC-MS feature table (m/z, intensity, retention time) before any candidate annotation has been produced — annotation metrics require a proposed-compound list, not peak intensities
- The goal is to evaluate the annotator's peak-matching or clustering stage (e.g. cluster-based filter reducing Assay 2 FP from 17598 to 1671) rather than the final entity-level annotation quality after diffusion prioritization
- Only one dataset is available without a reference compound set or with no per-assay TP/FP/FN ground truth — the metric cannot be computed and an alternative evaluation (e.g. only structural closeness) is needed

## Inputs

- Per-assay candidate annotation lists (e.g. top three KEGG compounds per LC-MS peak) from mWISE (FELLA or RClass diffusion, raw score or z-normalized score), xMSannotator, or mummichog
- Reference compound sets (true annotated compounds) for each assay
- Assay-to-reference mapping including reference false positives count (e.g. Assay 6: 1 ref FP, 74 references; Assay 1: 157 references)

## Outputs

- Per-assay TP, TN, FP, FN, ref FP, references counts
- Per-assay sensitivity, specificity, precision, NPV, F1
- Per-assay Tanimoto ratio comparing chemical structures of proposed vs original compounds
- Aggregated benchmark table mirroring the mWISE results tables (e.g. mWISE Assay 1: 144 TP, 11605 TN, 0 FP, 13 FN, sensitivity 0.92; FELLA raw-score Assay 6: 53 TP, 90 FP, 7852 TN, 21 FN, sensitivity 0.72, F1 0.49)

## How to apply

Load the per-assay annotation lists produced by the annotator (e.g. mWISE's top three candidates per peak from the FELLA-network + raw-score diffusion configuration, or xMSannotator output) and the reference compound sets for each assay. Derive TP/FP/FN/TN counts per assay from the annotator's candidate list and the reference set, and compute sensitivity, specificity, precision, F1, and NPV per assay exactly as in the paper's tables (e.g. mWISE Assay 6: 53 TP, 90 FP, 7852 TN, 21 FN, 1 ref FP, 74 references → sensitivity 0.72, F1 0.49; xMSannotator Assay 6: 54 TP, 369 FP, 7138 TN, 19 FN → F1 0.22; mummichog Assay 6: 27 TP, 84 FP, 8160 TN, 47 FN → F1 0.29). Separately compute a Tanimoto ratio between the chemical structures of proposed and original compounds per assay using the same structure-based similarity scheme reported for mWISE, since diffusion prioritization chemically brings proposed compounds closer to the originals (mWISE Assay 1 Tanimoto ratio 0.69; Assay 3 0.61). Aggregate the per-assay rows into a single table with columns TP, TN, FP, FN, ref FP, references, sensitivity, specificity, precision, NPV, F1, and Tanimoto ratio, and validate that recomputed F1/sensitivity values match the reported per-assay values (e.g. mWISE Assay 1 sensitivity 0.92 from 144 TP/13 FN/157 references; FELLA raw-score F1 0.49 for Assay 6; RClass raw-score F1 0.38 for Assay 6) to confirm the metric computation replicates the published benchmark.

## Related tools

- **mWISE** (Source of per-assay candidate annotation lists (top three KEGG compounds per LC-MS peak) that the metrics are computed against, using the FELLA or RClass diffusion configurations) — https://github.com/b2slab/mWISE
- **FELLA** (Provides the biological network used in the mWISE diffusion configuration whose raw-score annotations are one of the benchmark candidates (mWISE mean sensitivity 0.63 SD 0.07 vs xMSannotator 0.55 SD 0.19))
- **Diffu Stats** (R package used by mWISE to diffuse the input label vector in a given network, producing the raw/z-normalized diffusion scores that feed the prioritized candidate lists being benchmarked)
- **xMSannotator** (Baseline annotation algorithm whose per-assay TP/FP/FN counts and F1 (best 0.22, Assay 6) and sensitivity (0.55 SD 0.19) are computed and compared against mWISE in the same benchmark tables)
- **R** (Language/runtime in which the mWISE benchmark tables and diffusion-based annotation are implemented (mWISE is an R package; rmarkdown used for the vignette)) — https://github.com/b2slab/mWISE

## Examples

```
Rscript -e 'source("mWISE_metrics.R"); compute_metrics(assay="Assay 6", annotator="mWISE", config="FELLA_raw")'
```

## Evaluation signals

- Recomputed per-assay F1 matches the published table for the same configuration (e.g. mWISE Assay 6 F1 = 0.49; xMSannotator Assay 6 F1 = 0.22; mummichog Assay 6 F1 = 0.29)
- Recomputed per-assay sensitivity matches the reported value (e.g. mWISE matching sensitivity 0.92 for Assay 1 with 144 TP, 13 FN, 157 references; FELLA raw-score Assay 6 sensitivity 0.72)
- TP+FP+FN+TN consistency per assay: counts partition the annotated candidate/reference universe (e.g. mWISE Assay 1: 144 TP, 11605 TN, 0 FP, 13 FN; FELLA raw-score Assay 6: 53 TP, 90 FP, 7852 TN, 21 FN)
- Per-assay Tanimoto ratio falls in the expected structural-similarity range reported for mWISE (Assay 1: 0.69; Assay 3: 0.61), confirming many false positives are structurally close to the true compounds
- Cross-annotator comparison reproduces the paper's relative ordering: all four mWISE configurations outperform xMSannotator (mean sensitivity 0.63 SD 0.07 vs 0.55 SD 0.19; best F1 0.49 vs 0.22)

## Limitations

- The benchmark tables depend on the annotator's candidate list quality: mWISE lacks support for databases other than KEGG (limitation relative to xMSannotator which covers KEGG, HMDB, Lipid Maps, T3DB), so the metric comparison is confined to KEGG-compound annotations
- The Tanimoto ratio is only computed per-assay from the same structure-based similarity scheme reported for mWISE; comparing it against annotators whose chemical-structure proximity is not similarly reported (e.g. mummichog) is not directly supported by the metric scheme
- The metric computation is sensitive to the exact reference false-positive count (e.g. 1 ref FP for Assay 6, 34 for mummichog Assay 6) and the reference count per assay (74 vs 157), so mismatched assay-to-reference mappings will distort sensitivity/F1

## Evidence
