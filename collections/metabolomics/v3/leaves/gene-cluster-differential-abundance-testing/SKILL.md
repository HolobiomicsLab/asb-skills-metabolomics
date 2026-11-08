---
name: gene-cluster-differential-abundance-testing
description: Use when you have RPKM-normalized gene cluster abundance or expression
  tables plus sample grouping metadata and need to identify MGCs/GCFs significantly
  differentially abundant or expressed between conditions.
when_to_use_negative:
- Input is raw metagenomic/metatranscriptomic reads or unmapped data — mapping to
  gene cluster representatives via bigmap-map must happen first
- The goal is gene cluster prediction or grouping of BGCs into GCFs (antiSMASH/gutSMASH
  + bigmap-family territory), not differential testing between sample groups
- Only a single condition with no sample grouping metadata is available, so no between-group
  comparison can be performed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: BiG-MAP
  role: bigmap-analyse module implements CUSUM normalization (metagenomeSeq), ZIG
    and Kruskal-Wallis differential testing with FDR correction, and TSV/heatmap output
  repo: github.com/medema-group/BiG-MAP
- name: seaborn
  role: renders the heatmap visualization of significant gene cluster profiles
  repo: github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:7f579fa8e42cf9172605b31fa4b6b608d2c742db3b0b0f3517485e8818b3259d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - seaborn
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-cluster-differential-abundance-testing

## Summary

Statistical testing of metabolic/biosynthetic gene cluster (MGC/GCF) abundance or expression between sample groups using BiG-MAP.analyse, which applies cumulative sum scaling to RPKM values and zero-inflated Gaussian or Kruskal-Wallis tests with FDR correction.

## When to use

Use when you have RPKM-normalized gene cluster abundance or expression tables plus sample grouping metadata and need to identify MGCs/GCFs significantly differentially abundant or expressed between conditions.

## When NOT to use

- Input is raw metagenomic/metatranscriptomic reads or unmapped data — mapping to gene cluster representatives via bigmap-map must happen first
- The goal is gene cluster prediction or grouping of BGCs into GCFs (antiSMASH/gutSMASH + bigmap-family territory), not differential testing between sample groups
- Only a single condition with no sample grouping metadata is available, so no between-group comparison can be performed

## Inputs

- RPKM-normalized gene cluster abundance table (output of bigmap-map)
- RPKM-normalized gene cluster expression table (output of bigmap-map)
- Sample grouping metadata (condition labels per metagenomic/metatranscriptomic sample)

## Outputs

- Tab-separated (.tsv) differential abundance/expression test results with rows per gene cluster and FDR-adjusted statistics
- Seaborn heatmap of significant gene cluster profiles across sample groups

## How to apply

Load the RPKM-normalized gene cluster abundance and/or expression tables produced by bigmap-map/bigmap-family together with the sample grouping metadata. Normalize the RPKM values with cumulative sum scaling (cumNormStat-style CUSUM normalization as implemented in metagenomeSeq) to correct compositional bias and library-size differences between groups. For parametric comparison of grouped samples, fit the zero-inflated Gaussian mixture model via fitZIG on the scaled values; where the parametric route is inappropriate, run the nonparametric Kruskal-Wallis test on the same grouped samples. Correct test p-values for multiple testing with Benjamini-Hochberg FDR adjustment and flag significantly differentially abundant/expressed gene clusters. Write the differential results as tab-separated (.tsv) tables and render a heatmap of the significant gene cluster profiles with seaborn.

## Related tools

- **BiG-MAP** (bigmap-analyse module implements CUSUM normalization (metagenomeSeq), ZIG and Kruskal-Wallis differential testing with FDR correction, and TSV/heatmap output) — github.com/medema-group/BiG-MAP
- **seaborn** (renders the heatmap visualization of significant gene cluster profiles) — github.com/mwaskom/seaborn

## Evaluation signals

- Each emitted output matches the module's documented structure: a tab-separated TSV of differential results plus a heatmap file
- TSV rows correspond to gene clusters with FDR (Benjamini-Hochberg) adjusted statistics and group-level profile columns
- RPKM values were CUSUM-normalized before testing, not tested on raw RPKM values
- A parametric (fitZIG) or nonparametric (Kruskal-Wallis) test was chosen explicitly per analysis route
- Significant hits are plausible biologically — e.g. the oral microbiome case study recovered the muc operon (tooth decay) among differentially abundant clusters

## Limitations

- The tool has been only tested on Ubuntu
- Parametric ZIG modeling is inappropriate for some cluster/group combinations, requiring fallback to Kruskal-Wallis; statistical output is flagged as subject to expert review
- Differential calls depend on upstream redundancy filtering and GCF grouping quality (MASH 0.8 similarity cut-off; BiG-SCAPE recommended for more accurate redundancy filtering)

## Evidence

- [intro] Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [abstract] BiG-MAP profiles gene cluster abundance and expression and evaluates differential abundance: "to profile abundance and expression levels of gene clusters across metagenomic and metatranscriptomic data and evaluate their differential abundance and expression"
- [readme] README: bigmap-analyse performs differential analysis and visualizes output: [evidence span withheld — anchor status: external_doc]
- [readme] README: bigmap-map produces abundance and expression profiles consumed by the analyse module: [evidence span withheld — anchor status: external_doc]
- [readme] README: rationale for profiling MGC abundance and expression: [evidence span withheld — anchor status: external_doc]
