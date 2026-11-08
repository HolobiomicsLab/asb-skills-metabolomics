---
name: kruskal-wallis-nonparametric-group-comparison
description: Use when comparing gene cluster (BGC/MGC or GCF) abundance or expression
  across two or more sample groups from RPKM-normalized BiG-MAP tables, when the zero-inflated
  Gaussian parametric route is not appropriate for the data.
when_to_use_negative:
- Input is still raw metagenomic/metatranscriptomic reads rather than RPKM-normalized
  abundance/expression tables from bigmap-map — mapping and normalization must precede
  testing
- The data are well suited to the parametric zero-inflated Gaussian route (fitZIG),
  which BiG-MAP.analyse offers as the primary test; Kruskal-Wallis is the fallback
  for cases where the parametric route is inappropriate
- The goal is redundancy filtering, GCF grouping, or read mapping (use bigmap-family
  or bigmap-map) rather than between-group differential testing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP
  role: Hosts the bigmap-analyse module performing CUSUM normalization, ZIG/Kruskal-Wallis
    differential testing, and TSV/heatmap output
  repo: https://github.com/medema-group/BiG-MAP
- name: metagenomeSeq
  role: Provides the cumNormStat-style cumulative sum scaling used to normalize RPKM
    values before testing
- name: seaborn
  role: Renders the heatmap of significant differentially abundant/expressed gene
    clusters
  repo: https://github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:1b60cd8c9da652bfaf3da24e738d6700e3b600d88af0203d9844552c791cc844
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - metagenomeSeq
  - seaborn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kruskal-wallis-nonparametric-group-comparison

## Summary

Nonparametric differential-testing route of BiG-MAP.analyse: Kruskal-Wallis testing of cumulatively-sum-scaled (CUSUM) RPKM gene cluster abundance/expression values across sample groups, with Benjamini-Hochberg FDR correction, used where the zero-inflated Gaussian (ZIG) parametric route is inappropriate.

## When to use

Use when comparing gene cluster (BGC/MGC or GCF) abundance or expression across two or more sample groups from RPKM-normalized BiG-MAP tables, when the zero-inflated Gaussian parametric route is not appropriate for the data.

## When NOT to use

- Input is still raw metagenomic/metatranscriptomic reads rather than RPKM-normalized abundance/expression tables from bigmap-map — mapping and normalization must precede testing
- The data are well suited to the parametric zero-inflated Gaussian route (fitZIG), which BiG-MAP.analyse offers as the primary test; Kruskal-Wallis is the fallback for cases where the parametric route is inappropriate
- The goal is redundancy filtering, GCF grouping, or read mapping (use bigmap-family or bigmap-map) rather than between-group differential testing

## Inputs

- RPKM-normalized gene cluster abundance table (output of bigmap-map / bigmap-family)
- RPKM-normalized gene cluster expression table (metatranscriptomic, output of bigmap-map)
- Sample grouping metadata assigning each sample to a condition (e.g., healthy vs. caries, CD vs. UC)

## Outputs

- Tab-separated (.tsv) differential test results with gene cluster rows and FDR-adjusted statistics
- Seaborn heatmap of significant gene cluster profiles across sample groups

## How to apply

Load the RPKM-normalized gene cluster abundance and/or expression tables emitted by the bigmap-map and bigmap-family modules together with the sample grouping metadata (e.g., healthy vs. caries-associated oral samples, or CD vs. UC IBD samples). First apply cumulative sum scaling (cumNormStat-style CUSUM normalization as implemented in metagenomeSeq) to the RPKM values to correct for compositional bias and library-size differences between groups. Then run the nonparametric Kruskal-Wallis test on the CUSUM-scaled values for the grouped samples, reserving it for gene clusters where the parametric fitZIG route is inappropriate. Correct the resulting p-values for multiple testing with Benjamini-Hochberg FDR adjustment and flag significantly differentially abundant/expressed gene clusters. Emit the results as tab-separated (.tsv) tables with rows per gene cluster and render a seaborn heatmap of the significant gene clusters' group-level profiles.

## Related tools

- **BiG-MAP** (Hosts the bigmap-analyse module performing CUSUM normalization, ZIG/Kruskal-Wallis differential testing, and TSV/heatmap output) — https://github.com/medema-group/BiG-MAP
- **metagenomeSeq** (Provides the cumNormStat-style cumulative sum scaling used to normalize RPKM values before testing)
- **seaborn** (Renders the heatmap of significant differentially abundant/expressed gene clusters) — https://github.com/mwaskom/seaborn

## Evaluation signals

- Each output .tsv row corresponds to a gene cluster (BGC/MGC or GCF) and carries the test statistic with FDR (Benjamini-Hochberg)-adjusted p-value, matching the module's documented tabular output structure
- Confirm CUSUM normalization was applied to the RPKM values before testing (values corrected for compositional bias and library-size differences)
- Verify Kruskal-Wallis was applied to grouped samples where the parametric fitZIG route was inappropriate, and that p-values were FDR-corrected before significance flagging
- The rendered seaborn heatmap should show group-level profiles only for the significantly differentially abundant/expressed gene clusters
- Downstream sanity check: on the 96-sample healthy vs. caries oral cohort the module recovered 252 significantly differential gene clusters including the muc operon — results in a similar analysis should include known biology at plausible rates

## Limitations

- The tool has been only tested on Ubuntu, per the repository README
- Nonparametric testing is the fallback route; the module's default parametric analysis is the zero-inflated Gaussian (ZIG) mixture model via fitZIG, so Kruskal-Wallis results should be treated as subject to expert review of the statistical output
- Differential results depend on upstream redundancy filtering (MASH at default 0.8 similarity, optionally BiG-SCAPE) and GCF grouping; poorly filtered gene clusters can distort group comparisons

## Evidence

- [intro] Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] README module list: [evidence span withheld — anchor status: external_doc]
- [abstract] Oral microbiome application: "we analyzed 96 metagenomic samples from healthy and caries-associated human oral microbiome samples and identified 252 gene clusters"
