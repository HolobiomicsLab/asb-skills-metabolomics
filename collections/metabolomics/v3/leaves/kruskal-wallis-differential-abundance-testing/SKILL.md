---
name: kruskal-wallis-differential-abundance-testing
description: Use when per-sample MGC/GCF abundance or expression profiles (e.g. from
  bigmap-map) must be compared between conditions such as Crohn's disease vs healthy
  metagenomes/metatranscriptomes, and a nonparametric test is warranted.
when_to_use_negative:
- Input is raw reads or gene cluster sequences — bigmap-map must first quantify per-sample
  abundance/expression before any testing
- The goal is functional pathway profiling against reference databases (KEGG/MetaCyc)
  rather than gene-cluster-level abundance/expression testing
- Sample groups are so small or paired that a parametric alternative in bigmap-analyse
  is the intended choice
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP
  role: bigmap-analyse module performs the nonparametric (Kruskal-Wallis) differential
    abundance/expression testing and visualizes output
  repo: github.com/medema-group/BiG-MAP
- name: seaborn
  role: Statistical data visualization library used to visualize differential results
  repo: github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:1bd204501da324b00143273d1954739087e400a742342c92d7e9e6add7313720
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - seaborn
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kruskal-wallis-differential-abundance-testing

## Summary

Nonparametric statistical testing (Kruskal-Wallis) of per-sample gene cluster (MGC/GCF) abundance and expression profiles between conditions, as implemented in the bigmap-analyse module of the BiG-MAP pipeline.

## When to use

Use when per-sample MGC/GCF abundance or expression profiles (e.g. from bigmap-map) must be compared between conditions such as Crohn's disease vs healthy metagenomes/metatranscriptomes, and a nonparametric test is warranted.

## When NOT to use

- Input is raw reads or gene cluster sequences — bigmap-map must first quantify per-sample abundance/expression before any testing
- The goal is functional pathway profiling against reference databases (KEGG/MetaCyc) rather than gene-cluster-level abundance/expression testing
- Sample groups are so small or paired that a parametric alternative in bigmap-analyse is the intended choice

## Inputs

- Per-sample MGC/GCF abundance profiles from bigmap-map (mapped metagenomic reads)
- Per-sample MGC/GCF expression profiles from bigmap-map (mapped metatranscriptomic reads)
- Sample grouping metadata (e.g. Crohn's disease vs non-IBD healthy, from the PRJNA389280 accession list)

## Outputs

- Table of significant differentially abundant and differentially expressed gene clusters per Kruskal-Wallis test (TSV, e.g. bigmap_cd_differential_results.tsv)
- Visualizations of differential abundance/expression profiles across samples

## How to apply

Take the per-sample abundance and expression profiles that bigmap-map produced by mapping metagenomic/metatranscriptomic reads to representative gene cluster sequences and group annotations (GCFs from bigmap-family's MASH + BiG-SCAPE redundancy filtering). For each MGC/GCF, run the nonparametric Kruskal-Wallis test on the abundance vector and, separately, on the expression vector across sample groups (e.g. CD vs non-IBD controls). Choose parametric vs nonparametric testing as a decision point in bigmap-analyse; the nonparametric option is preferred when distributional assumptions are not defensible. Output the significant clusters to a results table (e.g. bigmap_cd_differential_results.tsv) and verify against known reference sets — for the Schirmer et al. IBD cohort (PRJNA389280) this is 11 differentially abundant and 8 differentially expressed gene clusters.

## Related tools

- **BiG-MAP** (bigmap-analyse module performs the nonparametric (Kruskal-Wallis) differential abundance/expression testing and visualizes output) — github.com/medema-group/BiG-MAP
- **seaborn** (Statistical data visualization library used to visualize differential results) — github.com/mwaskom/seaborn

## Evaluation signals

- The test is run separately on abundance (metagenomic) and expression (metatranscriptomic) profile vectors per MGC/GCF
- For the Schirmer et al. IBD cohort (PRJNA389280) with CGR gutSMASH references, the significant set should reproduce 11 differentially abundant and 8 differentially expressed gene clusters
- Results table is written as TSV (bigmap_cd_differential_results.tsv) with per-cluster test outcomes
- Clusters reported as significant correspond to those detected by Kruskal-Wallis, not by default parametric testing

## Limitations

- BiG-MAP has only been tested on Ubuntu (per README)
- Testing operates on profiles derived from read mapping, so ambiguous clusters that escape MinHash (0.8 similarity cut-off) and BiG-SCAPE redundancy filtering can confound results
- The article reports both parametric and nonparametric options; choosing the wrong one changes the significant set

## Evidence

- [intro] Nonparametric differential testing within BiG-MAP: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] bigmap-analyse performs differential analysis and visualization: [evidence span withheld — anchor status: external_doc]
- [abstract] BiG-MAP profiles and evaluates differential abundance/expression: "BiG-MAP, a bioinformatic tool to profile abundance and expression levels of gene clusters across metagenomic and metatranscriptomic data and evaluate their differential abundance and expression"
