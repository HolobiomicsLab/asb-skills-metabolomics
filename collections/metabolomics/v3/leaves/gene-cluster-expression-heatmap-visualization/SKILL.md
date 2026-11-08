---
name: gene-cluster-expression-heatmap-visualization
description: Use when BiG-MAP RPKM abundance/expression tables for grouped samples
  (e.g. healthy vs. caries, CD vs. UC) have been differentially tested and you need
  a heatmap of significant gene cluster profiles.
when_to_use_negative:
- Input is raw metagenomic/metatranscriptomic reads rather than RPKM-normalized gene
  cluster abundance/expression tables — mapping via bigmap-map must happen first
- The goal is per-sample abundance/expression profiling across samples without any
  group comparison, where no differential-testing heatmap is meaningful
- No sample grouping metadata exists, since the ZIG and Kruskal-Wallis comparisons
  and the group-level heatmap profiles both require defined sample groups
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: BiG-MAP
  role: bigmap-analyse module performs CUSUM normalization, ZIG/Kruskal-Wallis differential
    testing, and emits the TSV results and seaborn heatmap
  repo: github.com/medema-group/BiG-MAP
- name: seaborn
  role: Python visualization library used to render the heatmap of significant gene
    cluster profiles
  repo: github.com/mwaskom/seaborn
- name: metagenomeSeq
  role: provides the cumulative sum scaling (cumNormStat-style CUSUM) normalization
    applied to RPKM values before testing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:ede8047ca52399d04f343b0e5ce6afed0af74e5237949157fc9aecff83f953e6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - metagenomeSeq
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Gene cluster expression heatmap visualization

## Summary

Visualizes differentially abundant or expressed metabolic gene clusters (MGCs/GCFs) from BiG-MAP.analyse as a heatmap of group-level profiles, using seaborn, alongside tab-separated test results.

## When to use

Use when BiG-MAP RPKM abundance/expression tables for grouped samples (e.g. healthy vs. caries, CD vs. UC) have been differentially tested and you need a heatmap of significant gene cluster profiles.

## When NOT to use

- Input is raw metagenomic/metatranscriptomic reads rather than RPKM-normalized gene cluster abundance/expression tables — mapping via bigmap-map must happen first
- The goal is per-sample abundance/expression profiling across samples without any group comparison, where no differential-testing heatmap is meaningful
- No sample grouping metadata exists, since the ZIG and Kruskal-Wallis comparisons and the group-level heatmap profiles both require defined sample groups

## Inputs

- RPKM-normalized gene cluster abundance and/or expression tables from bigmap-map / bigmap-family
- Sample grouping metadata (condition/group labels, e.g. healthy vs. caries-associated samples)
- Differential testing results (ZIG or Kruskal-Wallis with FDR correction) identifying significant gene clusters

## Outputs

- Tab-separated (.tsv) differential abundance/expression result tables with gene cluster rows and FDR-adjusted statistics
- Heatmap image (seaborn/matplotlib) of significant gene cluster profiles across sample groups

## How to apply

Feed the RPKM-normalized gene cluster abundance/expression tables produced by bigmap-map/bigmap-family, plus sample grouping metadata, into the bigmap-analyse module, which first applies cumulative sum scaling (cumNormStat-style CUSUM normalization as implemented in metagenomeSeq) to correct compositional and library-size bias. Differential testing is then run via the zero-inflated Gaussian (ZIG) mixture model (fitZIG) or, where the parametric route is inappropriate, the Kruskal-Wallis test, with p-values FDR (Benjamini-Hochberg) corrected. The visualization step renders a heatmap of the significant gene cluster profiles with seaborn, a matplotlib-based Python visualization library, so reviewers can compare group-level abundance/expression of clusters such as the muc operon across conditions. Validation confirms each emitted output matches the module's documented structure: TSV tables with gene-cluster rows, FDR-adjusted statistics and group-level profile columns, plus the heatmap file; statistical outputs remain subject to expert_review.

## Related tools

- **BiG-MAP** (bigmap-analyse module performs CUSUM normalization, ZIG/Kruskal-Wallis differential testing, and emits the TSV results and seaborn heatmap) — github.com/medema-group/BiG-MAP
- **seaborn** (Python visualization library used to render the heatmap of significant gene cluster profiles) — github.com/mwaskom/seaborn
- **metagenomeSeq** (provides the cumulative sum scaling (cumNormStat-style CUSUM) normalization applied to RPKM values before testing)

## Evaluation signals

- Each emitted output matches the documented structure: tabular TSV differential results plus a heatmap file, with rows for gene clusters, FDR-adjusted statistics, and group-level profile columns
- Heatmap plots only gene clusters flagged significant after Benjamini-Hochberg FDR adjustment, not the full cluster set
- Input values to the heatmap reflect CUSUM-scaled RPKM values, not raw RPKM counts
- The statistical route used per cluster (parametric ZIG vs. nonparametric Kruskal-Wallis) is consistent with sample-group structure
- Statistical output passes expert_review before being treated as final

## Limitations

- BiG-MAP has only been tested on Ubuntu, per the repository README
- The statistical output is subject to expert_review; the workflow explicitly flags this as not fully machine-validated
- The heatmap covers only significant differentially abundant/expressed clusters, so non-significant cluster profiles are not visualized

## Evidence

- [readme] bigmap-analyse visualizes differential results: "Performs differential abundance/expression analysis and visualizes the output
   using bigmap-analyse"
- [intro] parametric or nonparametric differential testing: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [intro] output and visualize MGC/GCF profiles: "and uses these to output and visualize profiles of MGC or GCF abundance or expression levels across samples"
- [readme] seaborn is the heatmap rendering library: [evidence span withheld — anchor status: external_doc]
