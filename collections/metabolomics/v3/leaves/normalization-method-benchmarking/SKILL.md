---
name: normalization-method-benchmarking
description: Use when several normalization methods (e.g., tGAM/rGAM/rLOESS vs QC-RLSC,
  QC-RSC, SERFF, TIGER) must be compared on a batched metabolomics dataset to decide
  which recovers signal, using DAM overlap, p-value correlation, and replicate MAD
  concordance rather than QC-RSD reduction or DAM counts as.
when_to_use_negative:
- When the goal is simply to normalize a production dataset with one recommended method
  (use tGAM directly in Metanorm) rather than comparatively benchmarking multiple
  strategies
- When judging normalization success by reduction of QC-sample relative standard deviation
  (RSD) or by increased numbers of differentially abundant metabolites — the article
  explicitly flags both as benchmarking pitfalls
- When the dataset lacks QC and/or replicate samples, which the authors recommend
  including specifically to support normalization performance investigations
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: Metanorm
  role: R package providing batchwise robust normalization (tGAM default, rGAM, rLOESS,
    QC-RLSC, QC-RSC) plus pre/post and PCA visualization used to verify normalization
    performance during benchmarking
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime for Metanorm, fANCOVA, mgcv, and TIGER; used for Wilcoxon DAM testing,
    Spearman correlation computation, and Euclidean-distance clustering of methods
- name: fANCOVA
  role: Supplies loess.as used by the rLOESS normalization variant with Tukey biweight
    downweighting
- name: mgcv
  role: Supplies REML-fitted thin plate regression splines used by the tGAM and rGAM
    normalization variants
- name: SERFF
  role: Benchmarked multivariate normalization method run via its Shiny web interface
    (https://slfan.shinyapps.io/ShinySERRF/); expected to disagree strongly with the
    robust methods (29–41% joint DAM findings on BioHEART)
- name: TIGER
  role: Benchmarked multivariate normalization method run via its R package; second-highest
    DAM count on BioHEART after raw data, 40% MAD reduction on between-batch replicates
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:fae173c33085bd70114c654a826f692acda5cc43694e5edd1f4299cc7a0edfdd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - R
  - SERFF
  - TIGER
  - fANCOVA
  - mgcv
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# normalization-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks metabolomics normalization strategies (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER, unnormalized) on real batched datasets by comparing DAM overlap, p-value similarity, and between-batch replicate MAD concordance, explicitly avoiding QC-RSD or DAM-count proxies for normalization quality.

## When to use

Use when several normalization methods (e.g., tGAM/rGAM/rLOESS vs QC-RLSC, QC-RSC, SERFF, TIGER) must be compared on a batched metabolomics dataset to decide which best recovers signal, using DAM overlap, p-value correlation, and replicate MAD concordance rather than QC-RSD reduction or DAM counts as success proxies.

## When NOT to use

- When the goal is simply to normalize a production dataset with one recommended method (use tGAM directly in Metanorm) rather than comparatively benchmarking multiple strategies
- When judging normalization success by reduction of QC-sample relative standard deviation (RSD) or by increased numbers of differentially abundant metabolites — the article explicitly flags both as benchmarking pitfalls
- When the dataset lacks QC and/or replicate samples, which the authors recommend including specifically to support normalization performance investigations

## Inputs

- Raw metabolomics intensity matrix, unnormalized, log-transformed base e prior to normalization (e.g., BioHEART plasma: 1359 runs, 53 targeted metabolites, 15 batches; ENVIRONAGE urine: 393 runs, 269 metabolites, 6 batches)
- Batch vector assigning each sample run to its batch (batchwise normalization argument)
- Sample-type vector distinguishing QC from biological samples (metanorm.qc)
- Between-batch replicate-run intensity pairs (identical biological samples run in subsequent batches)
- Phenotype/group labels for DAM testing (e.g., hypertension vs non-hypertension; biological sex)

## Outputs

- One normalized intensity matrix per method (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER) plus the unnormalized baseline
- Per-method DAM lists from two-sided Wilcoxon rank sum tests at p < 0.05
- Pairwise DAM overlap percentage matrix across methods (e.g., bioheart_dam_overlap.csv, environage_dam_comparison.csv)
- Pairwise Spearman correlation matrix of p-values with Euclidean-distance complete-linkage clustering heatmap
- Per-method MAD of rescaled between-batch replicate differences and percentage MAD reduction versus unnormalized data (replicate_concordance_mad.tsv)

## How to apply

Log-transform the raw intensity matrix (base e) before normalization, then run batchwise normalization in the Metanorm R package with model='QC-RLSC' (loess, degree 2, LOOCV/GCV span selection), 'QC-RSC' (smooth.spline), 'rLOESS' (loess.as in fANCOVA with Tukey biweight downweighting), 'rGAM', and 'tGAM' (REML-fitted thin plate regression splines via mgcv, QCcheck=TRUE), running SERFF via its Shiny interface and TIGER via its R package in parallel. For each normalized matrix plus the raw unnormalized matrix, call two-sided Wilcoxon rank sum tests per metabolite (p < 0.05, e.g., hypertension n=390 vs n=612, or biological sex in ENVIRONAGE) to define DAMs, then compute pairwise DAM overlap percentages, pairwise Spearman correlations of p-values, and cluster methods by Euclidean distance of one-minus-correlation vectors with a complete-linkage heatmap. For replicate concordance, take pairs of identical biological samples run in subsequent batches, compute per-compound differences rescaled by mean intensity, and take the MAD per method, reporting the percentage reduction versus the unnormalized baseline. Interpret results neutrally: a higher raw-data DAM count is not evidence of better normalization, and the robust methods (tGAM/rGAM/rLOESS) are expected to cluster together while strongly disagreeing with SERFF, with tGAM proposed as the default choice.

## Related tools

- **Metanorm** (R package providing batchwise robust normalization (tGAM default, rGAM, rLOESS, QC-RLSC, QC-RSC) plus pre/post and PCA visualization used to verify normalization performance during benchmarking) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime for Metanorm, fANCOVA, mgcv, and TIGER; used for Wilcoxon DAM testing, Spearman correlation computation, and Euclidean-distance clustering of methods)
- **fANCOVA** (Supplies loess.as used by the rLOESS normalization variant with Tukey biweight downweighting)
- **mgcv** (Supplies REML-fitted thin plate regression splines used by the tGAM and rGAM normalization variants)
- **SERFF** (Benchmarked multivariate normalization method run via its Shiny web interface (https://slfan.shinyapps.io/ShinySERRF/); expected to disagree strongly with the robust methods (29–41% joint DAM findings on BioHEART))
- **TIGER** (Benchmarked multivariate normalization method run via its R package; second-highest DAM count on BioHEART after raw data, 40% MAD reduction on between-batch replicates)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormExample/")
```

## Evaluation signals

- Reproduced DAM overlap percentages fall within reported ranges: robust methods tGAM/rGAM/rLOESS ~92% joint findings on BioHEART; ENVIRONAGE robust 74–88%, QC-RLSC 47–51%, SERFF 68–70%, TIGER 64–67%
- Replicate MAD reductions versus unnormalized data match reported values: robust methods 49–52%, SERFF 41%, TIGER 40%, QC-RLSC/QC-RSC 18–22%
- Unnormalized raw data yields the highest DAM count (BioHEART), followed by TIGER and QC-R(L)SC — high raw DAM count must not be treated as better normalization
- Spearman p-value correlation heatmap with complete-linkage clustering reproduces the reported grouping (robust methods clustering together, SERFF/TIGER more distant)
- Expected output files (bioheart_dam_overlap.csv, environage_dam_comparison.csv, replicate_concordance_mad.tsv) are written with per-method overlap percentages and MAD/%-reduction columns

## Limitations

- Multiple testing correction was not applied in this benchmarking work (Wilcoxon at p < 0.05 only), so DAM overlap results are liberal; the article notes multiple testing correction is essential for downstream inference
- Multivariate outlier detection is not implemented (feature-level treatment only), and automated outlier detection/removal prior to normalization is nontrivial and not provided
- The benchmarked set is limited: broader sets of normalization methods, and richer simulation designs mimicking experimental data (resampling strategies, cross-metabolite correlations), remain future work
- Metanorm requires R >= 4.4.0 and QC vs biological sample types; the README recommends inspecting individual compound pre- vs post-normalization plots since apparent batch-effect removal in PCA can hide remaining drift in biological samples

## Evidence

- [readme] Robust package scope and method inventory: [evidence span withheld — anchor status: external_doc]
