---
name: zero-inflated-gaussian-model-fitting
description: Use when metagenomic/metatranscriptomic gene-cluster abundance or expression
  tables (e.g. RPKM counts from BiG-MAP mapping) have many zero-inflated values and
  you need parametric differential testing between grouped samples.
when_to_use_negative:
- Input is raw sequencing reads or unnormalized counts rather than RPKM-normalized
  abundance/expression tables — normalization must be done first.
- Samples have no group/condition metadata, since ZIG fitting and Kruskal-Wallis both
  compare grouped samples.
- Testing single-cluster hypotheses without multiple-testing context — FDR correction
  across gene clusters is part of this skill.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BiG-MAP
  role: bigmap-analyse module performs the differential abundance/expression analysis
    (ZIG or Kruskal-Wallis) and emits the results
  repo: github.com/medema-group/BiG-MAP
- name: seaborn
  role: renders the heatmap of significant gene cluster profiles from the differential-testing
    results
  repo: github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:b5c737138d95647a449c122374a9f87701f19b743551495a7a0c3ce491ba53b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# zero-inflated-gaussian-model-fitting

## Summary

Fits a zero-inflated Gaussian (ZIG) mixture model to cumulatively-sum-scaled (CUSUM) RPKM abundance/expression values of gene clusters via fitZIG (metagenomeSeq-style), as used by BiG-MAP.analyse to detect differentially abundant or expressed gene clusters between sample groups.

## When to use

Use when metagenomic/metatranscriptomic gene-cluster abundance or expression tables (e.g. RPKM counts from BiG-MAP mapping) have many zero-inflated values and you need parametric differential testing between grouped samples.

## When NOT to use

- Input is raw sequencing reads or unnormalized counts rather than RPKM-normalized abundance/expression tables — normalization must be done first.
- Samples have no group/condition metadata, since ZIG fitting and Kruskal-Wallis both compare grouped samples.
- Testing single-cluster hypotheses without multiple-testing context — FDR correction across gene clusters is part of this skill.

## Inputs

- RPKM-normalized gene cluster abundance table produced by BiG-MAP mapping (bigmap-map)
- RPKM-normalized gene cluster expression table (metatranscriptomic)
- Sample grouping metadata assigning samples to conditions (e.g. healthy vs. disease groups)

## Outputs

- Tab-separated (.tsv) table of differential abundance/expression test results with rows per gene cluster, FDR-adjusted statistics, and group-level profile columns
- Seaborn-rendered heatmap of significant gene cluster profiles across sample groups

## How to apply

First normalize the RPKM values with cumulative sum scaling (CUSUM-style normalization as implemented in metagenomeSeq) to correct for compositional bias and library-size differences between groups. Then fit the zero-inflated Gaussian mixture model on the scaled values via fitZIG for the parametric comparison of gene cluster abundance/expression between the grouped samples. Where the parametric route is inappropriate (e.g. non-normal residual behavior or insufficient model fit), fall back to the nonparametric Kruskal-Wallis test on the same grouped samples. Correct p-values for multiple testing using Benjamini-Hochberg FDR adjustment and flag significantly differentially abundant/expressed gene clusters. BiG-MAP.analyse consumes the RPKM-normalized gene cluster abundance and/or expression tables produced by bigmap-map/bigmap-family together with sample grouping metadata, so ensure those upstream outputs and the group assignments are aligned before fitting.

## Related tools

- **BiG-MAP** (bigmap-analyse module performs the differential abundance/expression analysis (ZIG or Kruskal-Wallis) and emits the results) — github.com/medema-group/BiG-MAP
- **seaborn** (renders the heatmap of significant gene cluster profiles from the differential-testing results) — github.com/mwaskom/seaborn

## Evaluation signals

- Each emitted output matches the module's documented output structure: a tabular TSV differential result plus a heatmap file.
- TSV rows are gene clusters with FDR-adjusted statistics and group-level profile columns.
- RPKM values are cumulative-sum-scaled (CUSUM) before fitting, not raw counts.
- Statistical output flagged for expert_review, e.g. spot-check that significant clusters (such as the muc operon in the oral caries case) are biologically consistent.

## Limitations

- The zero-inflated Gaussian parametric route is inappropriate for some clusters; the nonparametric Kruskal-Wallis test is the fallback, so mixed-model results must be interpreted carefully.
- BiG-MAP has been only tested on Ubuntu, per the repository README.
- Test p-values must be FDR-corrected across all gene clusters before significance calling; uncorrected comparisons overstate significance.

## Evidence

- [intro] Parametric vs nonparametric differential testing: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] bigmap-analyse output and visualization: [evidence span withheld — anchor status: external_doc]
- [abstract] Tool scope: differential abundance/expression between conditions: "BiG-MAP, a bioinformatic tool to profile abundance and expression levels of gene clusters across metagenomic and metatranscriptomic data and evaluate their differential abundance and expression under"
