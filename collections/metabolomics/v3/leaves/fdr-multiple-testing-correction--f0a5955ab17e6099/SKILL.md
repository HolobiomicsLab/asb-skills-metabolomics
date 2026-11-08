---
name: fdr-multiple-testing-correction
description: Decision support for identifying differentially abundant or expressed
  gene clusters across sample groups when thousands of clusters are tested simultaneously
  and raw p-values would otherwise inflate false positives.
when_to_use_negative:
- Input is raw metagenomic/metatranscriptomic reads rather than RPKM-normalized cluster
  tables — these must first pass through bigmap-map mapping and RPKM computation
- Only one sample group with no comparison design exists, so there is nothing to test
  differentially
- Per-cluster effect sizes are needed without multiple-testing context, e.g. single-gene
  targeted analysis where family-wise structure does not apply
tools:
- name: BiG-MAP
  role: The bigmap-analyse module performs CUSUM normalization, ZIG/Kruskal-Wallis
    testing, and FDR correction of differential abundance/expression results
  repo: github.com/medema-group/BiG-MAP
- name: seaborn
  role: Statistical data visualization library used to render heatmaps of the significant
    gene cluster profiles
  repo: github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:3c14ca8a8bff36cceb3eecebd84082b377bcc322896754e3de25a4c52551da1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - seaborn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fdr-multiple-testing-correction

## Summary

Corrects p-values from the differential abundance/expression tests run by BiG-MAP.analyse using FDR (Benjamini-Hochberg) adjustment, so that significant gene clusters are flagged while controlling the false discovery rate across the many tested BGC/MGC families.

## When to use

Decision support for identifying differentially abundant or expressed gene clusters across sample groups when thousands of clusters are tested simultaneously and raw p-values would otherwise inflate false positives.

## When NOT to use

- Input is raw metagenomic/metatranscriptomic reads rather than RPKM-normalized cluster tables — these must first pass through bigmap-map mapping and RPKM computation
- Only one sample group with no comparison design exists, so there is nothing to test differentially
- Per-cluster effect sizes are needed without multiple-testing context, e.g. single-gene targeted analysis where family-wise structure does not apply

## Inputs

- RPKM-normalized gene cluster abundance table from bigmap-map
- RPKM-normalized gene cluster expression table from bigmap-map (metatranscriptomic)
- Sample grouping metadata (e.g. healthy vs caries-associated oral samples, CD vs UC IBD samples)

## Outputs

- Tab-separated (.tsv) differential abundance/expression results with rows per gene cluster and FDR-adjusted statistics
- Seaborn-rendered heatmap of significant gene cluster abundance/expression profiles across sample groups

## How to apply

Load the RPKM-normalized gene cluster abundance and/or expression tables produced by bigmap-map/bigmap-family together with the sample grouping metadata (e.g. healthy vs caries, Crohn's disease vs ulcerative colitis). First apply cumulative sum scaling (cumNormStat-style CUSUM normalization as implemented in metagenomeSeq) to the RPKM values to correct compositional bias and library-size differences. Then run the zero-inflated Gaussian (ZIG) mixture model via fitZIG as the parametric route, and the Kruskal-Wallis test as the nonparametric route for clusters where the parametric model is inappropriate. Collect the p-values from both tests and adjust them with the Benjamini-Hochberg FDR procedure before flagging significant clusters. Emit the adjusted statistics as tab-separated (.tsv) tables with one row per gene cluster and render heatmaps of significant cluster profiles with seaborn.

## Related tools

- **BiG-MAP** (The bigmap-analyse module performs CUSUM normalization, ZIG/Kruskal-Wallis testing, and FDR correction of differential abundance/expression results) — github.com/medema-group/BiG-MAP
- **seaborn** (Statistical data visualization library used to render heatmaps of the significant gene cluster profiles) — github.com/mwaskom/seaborn

## Evaluation signals

- Each emitted output matches the module's documented structure: a tabular TSV differential results file and a heatmap file
- TSV rows correspond one-to-one to gene clusters from the bigmap-map/bigmap-family input tables, with FDR-adjusted statistic columns
- The set of flagged significant clusters changes monotonically with the FDR threshold (a cluster significant at q<0.05 remains significant at q<0.10)
- Kruskal-Wallis and ZIG routes cover complementary clusters: clusters flagged parametric-inappropriate fall back to the nonparametric test rather than being dropped
- In the published oral microbiome application, differential testing recovered a plausible positive control — the muc operon associated with tooth decay

## Limitations

- The parametric ZIG route is inappropriate for some clusters; the nonparametric Kruskal-Wallis fallback must be used there
- The statistical output structure is subject to expert_review validation in the source card
- BiG-MAP has only been tested on Ubuntu, per the repository README
- Significant abundance does not imply biosynthetic activity: the reuterin cluster case was enriched but not exclusively found in healthy samples, requiring metabolomic follow-up

## Evidence

- [intro] parametric or nonparametric testing: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] analyse module role: [evidence span withheld — anchor status: external_doc]
- [abstract] differential abundance yields validated biology: "Among them, we found the muc operon, a gene cluster known to be associated with tooth decay."
