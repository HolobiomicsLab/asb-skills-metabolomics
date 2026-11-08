---
name: differential-abundance-testing
description: Use when one or more normalized metabolomics intensity matrices (rows
  = metabolites/features, columns = sample runs) must be compared for a binary phenotype
  contrast to detect DAMs and assess normalization-method agreement.
when_to_use_negative:
- You require multiplicity-controlled inference (FDR-adjusted) — the source workflow
  calls DAMs at uncorrected p < 0.05, and the article states multiple testing correction
  was not in scope though it is essential for downstream statistical inference.
- Your goal is to judge normalization quality by the raw number of DAMs found — the
  article explicitly warns against using increased numbers of differentially abundant
  metabolites as a proxy for good normalization (raw data yielded the highest DAM
  count).
- You only have a QC-RSD-filtered or otherwise pre-reduced matrix and want to benchmark
  filtering strategies rather than test differential abundance — the article flags
  RSD-in-QC filtering as potentially invalid when QC and biological samples differ.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: Metanorm R package
  role: Produces the per-method normalized matrices (tGAM, rGAM, rLOESS, QC-RLSC,
    QC-RSC, batchwise) that serve as the inputs to the Wilcoxon DAM testing; its plotPCA
    diagnostic verifies batch effects are reduced before testing
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment for the two-sided Wilcoxon rank sum tests, Spearman correlations,
    Euclidean-distance computation, and complete-linkage clustering/heatmap construction
- name: SERFF
  role: Alternative multivariate normalization pipeline whose Shiny-interface output
    provides an additional normalized matrix for the DAM-overlap comparison (failed
    on the 618-run x 8913-feature FAME dataset)
- name: TIGER
  role: Alternative multivariate normalization method (R package) whose normalized
    matrix is included in the pairwise DAM-overlap and p-value-similarity benchmarking
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:b5111c6af549fe5e70df5be275f17516428452628ad6e1c4d33ab9fe6a2eca6f
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
  - Metanorm R package
  - R
  - SERFF
  - TIGER
  - fANCOVA
  - mgcv
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# differential-abundance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differentially abundant metabolite (DAM) detection on normalized metabolomics matrices using two-sided Wilcoxon rank sum tests at p < 0.05, plus pairwise DAM-overlap and p-value-similarity benchmarking across normalization methods. Used to quantify how the choice of normalization (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER, raw) changes downstream biological calls.

## When to use

Use when one or more normalized metabolomics intensity matrices (rows = metabolites/features, columns = sample runs) must be compared for a binary phenotype contrast to detect DAMs and assess normalization-method agreement.

## When NOT to use

- You require multiplicity-controlled inference (FDR-adjusted) — the source workflow calls DAMs at uncorrected p < 0.05, and the article states multiple testing correction was not in scope though it is essential for downstream statistical inference.
- Your goal is to judge normalization quality by the raw number of DAMs found — the article explicitly warns against using increased numbers of differentially abundant metabolites as a proxy for good normalization (raw data yielded the highest DAM count).
- You only have a QC-RSD-filtered or otherwise pre-reduced matrix and want to benchmark filtering strategies rather than test differential abundance — the article flags RSD-in-QC filtering as potentially invalid when QC and biological samples differ.

## Inputs

- Normalized metabolomics intensity matrices (one per normalization method: tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER, plus the raw unnormalized matrix)
- Batch vector assigning each sample run to a batch (e.g., 15 batches for BioHEART, 6 for ENVIRONAGE)
- Binary phenotype/group labels per biological sample (e.g., hypertension status n=390 vs 612; biological sex; healthy-weight n=337 vs overweight/obesity n=99)
- Vector of sample types (QC vs non-QC) used upstream by Metanorm normalization

## Outputs

- Per-method DAM lists (metabolites/features with two-sided Wilcoxon p < 0.05)
- Pairwise DAM overlap percentage tables (e.g., bioheart_dam_overlap.csv, environage_dam_comparison.csv)
- Pairwise Spearman correlations of p-value vectors across methods
- Euclidean-distance (one-minus-correlation) heatmap with complete-linkage hierarchical clustering of methods
- Explicit failure record when a method cannot be run (e.g., SERFF server capacity exceeded on FAME)

## How to apply

For each normalized (or raw unnormalized) intensity matrix — e.g., log(e)-transformed BioHEART plasma data with 1359 runs and 53 targeted metabolites, ENVIRONAGE urine (393 runs, 269 metabolites), or FAME saliva (618 runs, 8913 features) — run two-sided Wilcoxon rank sum tests per metabolite between the two biological groups (e.g., hypertension n=390 vs n=612; healthy-weight n=337 vs overweight/obesity n=99) and call DAMs at p < 0.05. Then compute pairwise DAM overlap percentages across methods, pairwise Spearman correlations of p-value vectors, and similarity as Euclidean distances of one-minus-correlation vectors with complete-linkage hierarchical clustering rendered as a heatmap. The rationale is that normalization choice propagates into DAM calls: in the source study, robust methods (tGAM/rGAM/rLOESS) agreed at 92% joint findings on BioHEART, 74–88% on ENVIRONAGE, and 79–87% on FAME, while QC-RLSC agreed at only 47–51% (ENVIRONAGE) and 55–59% (FAME), and unnormalized raw data inflated the DAM count. Note that the article explicitly applied no multiple-testing correction and flags such correction as essential for downstream statistical inference in general.

## Related tools

- **Metanorm R package** (Produces the per-method normalized matrices (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, batchwise) that serve as the inputs to the Wilcoxon DAM testing; its plotPCA diagnostic verifies batch effects are reduced before testing) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment for the two-sided Wilcoxon rank sum tests, Spearman correlations, Euclidean-distance computation, and complete-linkage clustering/heatmap construction)
- **SERFF** (Alternative multivariate normalization pipeline whose Shiny-interface output provides an additional normalized matrix for the DAM-overlap comparison (failed on the 618-run x 8913-feature FAME dataset))
- **TIGER** (Alternative multivariate normalization method (R package) whose normalized matrix is included in the pairwise DAM-overlap and p-value-similarity benchmarking)

## Examples

```
wilcox.test(intensity ~ group, data = df, alternative = "two.sided")  # per metabolite; call DAM at p < 0.05
```

## Evaluation signals

- Robust-method agreement reproduces reported ranges: tGAM/rGAM/rLOESS 92% joint DAM findings on BioHEART, 74–88% on ENVIRONAGE, 79–87% on FAME
- QC-RLSC agreement falls in the reported weaker ranges: 47–51% (ENVIRONAGE) and 55–59% (FAME); SERFF 68–70% and TIGER 64–67% on ENVIRONAGE
- Raw unnormalized data yields the highest DAM count (BioHEART), followed by TIGER and QC-R(L)SC — raw inflation is the expected direction, not a sign of better performance
- SERFF failure on FAME must appear as an explicit record in the output table (dataset exceeded server capacity), not as missing rows
- Heatmap is generated with complete-linkage clustering of one-minus-Spearman-correlation Euclidean distances across methods

## Limitations

- DAM calls at p < 0.05 are uncorrected for multiple testing — the article states multiple testing correction was not in scope while being essential for downstream statistical inference
- DAM count alone must not be interpreted as normalization quality; the article warns against using increased numbers of differentially abundant metabolites as a benchmarking proxy
- Differences between QC and biological samples may invalidate QC-based filtering steps (e.g., RSD in QC) applied before testing
- SERFF could not process the FAME dataset (618 runs x 8913 features) through its Shiny interface, so large-scale comparisons may lack that method

## Evidence

- [readme] We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples.: [evidence span withheld — anchor status: external_doc]
