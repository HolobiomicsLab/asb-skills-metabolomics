---
name: fitzig-differential-abundance-testing
description: Use when you have a per-sample gene cluster abundance/coverage matrix
  (e.g., RPKM or core coverage from bigmap-map) plus phenotype labels (healthy vs
  caries) and need to identify significantly differentially abundant MGCs/GCFs.
when_to_use_negative:
- Input is raw metagenomic reads or unprocessed FASTQ files — mapping and abundance
  profiling via bigmap-map must be done first; this skill only operates on abundance/coverage
  tables.
- Samples lack phenotype labels or there is only one condition — differential testing
  requires at least two groups (healthy vs disease) to compare.
- The goal is gene cluster discovery/prediction rather than comparison — use gutSMASH/antiSMASH
  and bigmap-family instead; this skill does not mine or group clusters.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: bigmap-analyse
  role: BiG-MAP module that performs differential abundance/expression analysis (fitZIG
    parametric or Kruskal-Wallis nonparametric) on mapped abundance profiles and visualizes
    the output
  repo: https://github.com/medema-group/BiG-MAP
- name: BiG-MAP
  role: Parent pipeline providing the abundance and coverage profiles that feed the
    differential testing step
  repo: https://github.com/medema-group/BiG-MAP
- name: seaborn
  role: Statistical visualization library used to plot differential abundance/expression
    results produced by bigmap-analyse
  repo: https://github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:ea0a3016e0167810fd330ba5a891b848d18e5cdbf38fc2a56cc705a1fb2c3668
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - antiSMASH
  - bigmap-analyse
  - bigmap-map
  - gutSMASH
  - ncbi/sra-tools
  - seaborn
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# fitzig-differential-abundance-testing

## Summary

Statistical differential-abundance testing of gene cluster (MGC/GCF) abundance or expression profiles between phenotypes, using BiG-MAP's bigmap-analyse module with parametric (fitZIG zero-inflated) or nonparametric (Kruskal-Wallis) tests, as demonstrated on 96 healthy vs caries oral metagenomes.

## When to use

Use when you have a per-sample gene cluster abundance/coverage matrix (e.g., RPKM or core coverage from bigmap-map) plus phenotype labels (healthy vs caries) and need to identify significantly differentially abundant MGCs/GCFs.

## When NOT to use

- Input is raw metagenomic reads or unprocessed FASTQ files — mapping and abundance profiling via bigmap-map must be done first; this skill only operates on abundance/coverage tables.
- Samples lack phenotype labels or there is only one condition — differential testing requires at least two groups (healthy vs disease) to compare.
- The goal is gene cluster discovery/prediction rather than comparison — use gutSMASH/antiSMASH and bigmap-family instead; this skill does not mine or group clusters.

## Inputs

- Per-sample MGC/GCF abundance matrix (RPKM values) from bigmap-map
- Per-sample core coverage values per gene cluster from bigmap-map
- Phenotype metadata assigning each of the 96 oral metagenome samples to healthy or caries-associated groups
- GCF definitions (BiG-MAP.GCF.json / BiG-MAP.GCF.bed) from bigmap-family

## Outputs

- Differential abundance table (e.g., differential_abundance_results.tsv) with test statistics and P values per GCF
- Group-level summary statistics: mean RPKM per phenotype, mean core coverage, filtered mean coverage after the 0.5 threshold
- Verification summary (e.g., pdu_operon_verification.json) comparing computed values to reported figures
- Visualization of differential abundance profiles (rendered with seaborn/matplotlib)

## How to apply

Run bigmap-analyse on the abundance profiles produced by bigmap-map, choosing the model per analysis: the parametric fitZIG zero-inflated model was used for the Streptococcus mutans 14D muc operon comparison (explicitly not Kruskal-Wallis), while the nonparametric Kruskal-Wallis test was used for the Streptococcus sp. F0442 pdu-like operon (P = 0.00049). Because low-coverage samples distort group means, filter to samples with core coverage >= 0.5 before computing filtered mean coverage (e.g., muc operon: 8 healthy and 6 disease samples retained, means 0.848 vs 0.997 versus unfiltered 0.274 vs 0.161). Report group means, P values, and counts of differentially abundant clusters (246 whole set / 173 core in the pdu analysis). Verify results against independent subsets (e.g., the Aleti et al. PRJNA478018 subset showed no significant difference) and record outputs such as differential_abundance_results.tsv or pdu_operon_verification.json.

## Related tools

- **bigmap-analyse** (BiG-MAP module that performs differential abundance/expression analysis (fitZIG parametric or Kruskal-Wallis nonparametric) on mapped abundance profiles and visualizes the output) — https://github.com/medema-group/BiG-MAP
- **BiG-MAP** (Parent pipeline providing the abundance and coverage profiles that feed the differential testing step) — https://github.com/medema-group/BiG-MAP
- **seaborn** (Statistical visualization library used to plot differential abundance/expression results produced by bigmap-analyse) — https://github.com/mwaskom/seaborn

## Examples

```
conda activate bigmap; bigmap-analyse -h
```

## Evaluation signals

- Muc operon (S. mutans 14D) reproduces fitZIG-significant enrichment with unfiltered mean core coverages of 0.274 (healthy) vs 0.161 (caries).
- After filtering to samples with core coverage >= 0.5 (8 healthy, 6 disease), filtered mean coverages match reported 0.848 and 0.997 with minimal fold change between group averages.
- pdu-like operon (Streptococcus sp. F0442) reproduces healthy 5.62 vs diseased 3.38 RPKM with Kruskal-Wallis P = 0.00049.
- Counts of differentially abundant clusters match reported 246 (whole set) and 173 (core).
- Independent subset check: the smaller Aleti et al. (PRJNA478018) subset shows no significant difference for the muc operon.

## Limitations

- Choice of statistical model changes conclusions: the muc operon was fitZIG-significant but not Kruskal-Wallis-significant, so the parametric and nonparametric tests are not interchangeable.
- Samples with core coverage below 0.5 must be excluded before computing filtered mean coverage (13 of 34 healthy samples fell below the threshold in the pdu analysis); including them masks true group differences.
- BiG-MAP has only been tested on Ubuntu, per the repository README.
- Replication on smaller independent datasets (e.g., PRJNA478018) may fail to reach significance even when the main cohort does.

## Evidence

- [intro] Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [abstract] we analyzed 96 metagenomic samples from healthy and caries-associated human oral microbiome samples and identified 252 gene clusters: "we analyzed 96 metagenomic samples from healthy and caries-associated human oral microbiome samples and identified 252 gene clusters, including unreported ones, that were significantly more abundant"
- [readme] 4) Performs differential abundance/expression analysis and visualizes the output using bigmap-analyse: "4) Performs differential abundance/expression analysis and visualizes the output
   using bigmap-analyse"
- [other] The muc operon from Streptococcus mutans 14D was differentially abundant between healthy and disease samples under the fitZIG model (not Kruskal-Wallis): [evidence span withheld — anchor status: not_found]
- [other] healthy samples averaging 5.62 RPKM versus 3.38 for diseased samples (Kruskal-Wallis P = 0.00049): [evidence span withheld — anchor status: not_found]
