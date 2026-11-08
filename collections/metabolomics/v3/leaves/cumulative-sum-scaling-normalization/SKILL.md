---
name: cumulative-sum-scaling-normalization
description: Use when BiG-MAP RPKM-normalized gene cluster (MGC/GCF) abundance or
  expression tables plus sample grouping metadata must be compositionally normalized
  and tested for differential abundance/expression between conditions.
when_to_use_negative:
- Input is raw read counts or mapping output that has not yet been RPKM-normalized
  by bigmap-map — normalize the abundance profile first
- Input tables are already CSS-normalized; reapplying cumulative sum scaling would
  double-normalize the compositional bias correction
- The goal is functional profiling of pathways rather than gene cluster (MGC/GCF)
  abundance — BiG-MAP explicitly positions itself against pathway tools like HUMAnN2,
  MetaPath, FMAP and Metatrans
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP
  role: bigmap-analyse module performs the CSS normalization, ZIG/Kruskal-Wallis differential
    testing and FDR correction
  repo: github.com/medema-group/BiG-MAP
- name: seaborn
  role: renders the heatmap of significant gene cluster profiles from the differential
    test output
  repo: mwaskom/seaborn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:61634a629fddce84245a66a60c3fad0adaf3f2a52cc75c665cb0a2a80b9ed0f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - seaborn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cumulative-sum-scaling-normalization

## Summary

Normalizes RPKM gene cluster abundance/expression values from BiG-MAP using cumulative sum scaling (CSS, cumNormStat/metagenomeSeq-style) and then flags differentially abundant or expressed gene clusters between sample groups via zero-inflated Gaussian (ZIG) or Kruskal-Wallis tests with FDR correction.

## When to use

Use when BiG-MAP RPKM-normalized gene cluster (MGC/GCF) abundance or expression tables plus sample grouping metadata must be compositionally normalized and tested for differential abundance/expression between conditions.

## When NOT to use

- Input is raw read counts or mapping output that has not yet been RPKM-normalized by bigmap-map — normalize the abundance profile first
- Input tables are already CSS-normalized; reapplying cumulative sum scaling would double-normalize the compositional bias correction
- The goal is functional profiling of pathways rather than gene cluster (MGC/GCF) abundance — BiG-MAP explicitly positions itself against pathway tools like HUMAnN2, MetaPath, FMAP and Metatrans

## Inputs

- RPKM-normalized gene cluster abundance table from bigmap-map
- RPKM-normalized gene cluster expression table from metatranscriptomic samples (bigmap-map)
- Sample grouping/metadata table defining the condition groups (e.g. healthy vs. caries, CD vs. UC)
- GCF/HGF dictionaries from bigmap-family (e.g. BiG-MAP.GCF.json) defining the analysis units

## Outputs

- Tab-separated (.tsv) differential abundance/expression test results with rows per gene cluster, FDR-adjusted statistics, and group-level profile columns
- Heatmap image (seaborn) of significant gene cluster profiles across samples

## How to apply

Load the RPKM-normalized gene cluster abundance and/or expression tables produced by bigmap-map (and family grouping from bigmap-family, where BiG-SCAPE-defined GCFs are the analysis units) together with the sample grouping metadata. Apply cumulative sum scaling (cumNormStat-style CUSUM normalization, as implemented in metagenomeSeq) to correct for compositional bias and library-size differences between groups. For parametric comparison, fit the zero-inflated Gaussian (ZIG) mixture model via fitZIG on the scaled values; where the parametric route is inappropriate, run the nonparametric Kruskal-Wallis test on the same grouped samples. Correct p-values for multiple testing with FDR (Benjamini-Hochberg) adjustment and flag significantly differentially abundant/expressed gene clusters. Write the differential results as tab-separated (.tsv) tables and render a heatmap of significant gene cluster profiles with seaborn.

## Related tools

- **BiG-MAP** (bigmap-analyse module performs the CSS normalization, ZIG/Kruskal-Wallis differential testing and FDR correction) — github.com/medema-group/BiG-MAP
- **seaborn** (renders the heatmap of significant gene cluster profiles from the differential test output) — mwaskom/seaborn

## Evaluation signals

- Each emitted output matches the module's documented structure: tabular .tsv differential results plus a heatmap file
- Every tested gene cluster row carries an FDR (Benjamini-Hochberg)-adjusted statistic and group-level profile columns
- CSS scaling factors have been applied to all samples, correcting library-size and compositional differences before testing
- Parametric route used fitZIG and the nonparametric route used Kruskal-Wallis on the same grouped samples, with significance defined only after FDR adjustment
- Heatmap rows correspond to the flagged significant gene clusters, not the full input table

## Limitations

- The zero-inflated Gaussian (ZIG) model is not appropriate for all data, requiring a nonparametric Kruskal-Wallis fallback for those cases
- BiG-MAP has been only tested on Ubuntu, and results are subject to expert_review of the statistical output
- Upstream GCF definition depends on redundancy filtering (MASH default 0.8 similarity cut-off, optionally BiG-SCAPE); inaccurate filtering propagates into the tested clusters

## Evidence

- [other] BiG-MAP.analyse normalizes RPKM values with cumulative sum scaling, detects differentially abundant/expressed gene clusters via either the zero-inflated Gaussian (ZIG) mixture model: [evidence span withheld — anchor status: not_found]
- [intro] Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] Performs differential abundance/expression analysis and visualizes the output using bigmap-analyse: "4) Performs differential abundance/expression analysis and visualizes the output
   using bigmap-analyse"
