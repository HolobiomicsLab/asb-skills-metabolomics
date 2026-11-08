---
name: phenotype-differential-enrichment-analysis
description: Use when you have a feature quantification table plus phenotype metadata
  (e.g. 25 healthy vs 24 caries samples) and need to identify which molecular families
  or gene clusters are differentially abundant between the two conditions.
when_to_use_negative:
- Input is already a single-family or single-metabolite table with no phenotype grouping
  — set-based enrichment scoring is unnecessary; use per-feature differential testing
  instead
- Samples lack phenotype/condition metadata — PALS scoring requires phenotype labels
  to compute set activity per condition
- The goal is BGC discovery from genomes rather than differential abundance profiling
  across metagenomic samples — use antiSMASH/gutSMASH directly instead of the enrichment
  workflow
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: BiG-MAP
  role: Downloads sample data (bigmap-download), profiles gene cluster abundance/expression
    via read mapping, and computes differential abundance with parametric or nonparametric
    tests (bigmap-analyse)
  repo: github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Groups BGCs into gene cluster families (GCFs) so enrichment is tested at family
    level rather than per single cluster
- name: gutSMASH
  role: Predicts specialized-primary-metabolism metabolic gene clusters that serve
    as the features whose differential abundance is tested
  repo: victoriapascal/gutsmash
- name: ncbi/sra-tools
  role: Underlying SRA download machinery used by bigmap-download to fetch .SRA files
    and convert them to .fastq via fastq-dump
  repo: ncbi/sra-tools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:e7c742ce776a2af255eedd6d978122fde7b6d0c6614150d1f35465f1b7063d96
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - gutSMASH
  - ncbi/sra-tools
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# phenotype-differential-enrichment-analysis

## Summary

A set-based differential enrichment workflow that tests whether molecular families (from Feature-Based Molecular Networking) or metabolic gene clusters are enriched in one phenotype versus another, e.g. healthy vs caries-associated oral samples, using PALS activity scoring and BiG-MAP differential abundance testing.

## When to use

Use when you have a feature quantification table plus phenotype metadata (e.g. 25 healthy vs 24 caries samples) and need to identify which molecular families or gene clusters are differentially abundant between the two conditions.

## When NOT to use

- Input is already a single-family or single-metabolite table with no phenotype grouping — set-based enrichment scoring is unnecessary; use per-feature differential testing instead
- Samples lack phenotype/condition metadata — PALS scoring requires phenotype labels to compute set activity per condition
- The goal is BGC discovery from genomes rather than differential abundance profiling across metagenomic samples — use antiSMASH/gutSMASH directly instead of the enrichment workflow

## Inputs

- Processed MGF spectra file from a GNPS-MassIVE metabolomics accession (e.g. MSV000081832)
- Feature quantification table (feature abundance per sample) from FBMN
- Sample metadata / phenotype table mapping samples to conditions (e.g. 25 healthy vs 24 caries samples)
- BiG-MAP.GCF.fna and gene cluster BED files (for gene-cluster differential abundance analysis)

## Outputs

- pals_activity_scores.csv — full per-family PALS activity score table for each phenotype
- acrolein_family_summary.csv — enrichment summary for (poly)acrolein-annotated molecular families
- Differential abundance/expression statistics per gene cluster or gene cluster family from bigmap-analyse

## How to apply

Download the processed MGF spectra, feature quantification table, and sample metadata from the GNPS-MassIVE accession using the BiG-MAP `bigmap-download` entry point. Run Feature-Based Molecular Networking (FBMN) on the MGF spectra and feature table to cluster features into molecular families by spectral fragmentation similarity, then run PALS (Pathway Activity Level Scoring) on the feature table with the phenotype metadata to compute set-based activity scores per molecular family. Rank families by the PALS activity score for the target phenotype (e.g. healthy) and filter for member features carrying candidate annotations (e.g. polymer-like (poly)acrolein). For gene-cluster-level enrichment, use BiG-MAP's analyse module, which maps reads to representative gene clusters and computes differential abundance with parametric or nonparametric tests. Write the full per-family score table (pals_activity_scores.csv) and the annotation-filtered enrichment summary (acrolein_family_summary.csv); validation requires the candidate-annotated family to score as enriched in the reported phenotype, consistent with the paper's finding that (poly)acrolein-like masses came from samples in which a reuterin biosynthetic gene cluster was enriched.

## Related tools

- **BiG-MAP** (Downloads sample data (bigmap-download), profiles gene cluster abundance/expression via read mapping, and computes differential abundance with parametric or nonparametric tests (bigmap-analyse)) — github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Groups BGCs into gene cluster families (GCFs) so enrichment is tested at family level rather than per single cluster)
- **gutSMASH** (Predicts specialized-primary-metabolism metabolic gene clusters that serve as the features whose differential abundance is tested) — victoriapascal/gutsmash
- **ncbi/sra-tools** (Underlying SRA download machinery used by bigmap-download to fetch .SRA files and convert them to .fastq via fastq-dump) — ncbi/sra-tools

## Examples

```
bigmap-download -A Acc_list.txt -O /usr001/fastq/schirmer/
```

## Evaluation signals

- pals_activity_scores.csv contains one activity score per molecular family for each phenotype and ranks families by the target phenotype's score
- The (poly)acrolein-annotated molecular family scores as enriched in healthy samples, matching the paper's reported finding
- acrolein_family_summary.csv identifies a differentially expressed molecular family with three candidate metabolites annotated as polyacrolein-like
- BiG-MAP differential abundance output shows the reuterin biosynthetic gene cluster from a Streptococcus strain enriched (not exclusively) in healthy samples, consistent with the metabolomics signal
- Sample counts in the analysis match the metadata: 25 healthy and 24 caries samples

## Limitations

- BiG-MAP has only been tested on Ubuntu; other OS support is not documented
- Enrichment is not exclusive — the reuterin BGC was 'enriched but not exclusively found in healthy samples', so phenotype enrichment does not imply phenotype specificity
- bigmap-download depends on shotgun data being present in the SRA repository under the given accession list
- The second (more accurate) redundancy filtering step in bigmap-family requires a separate BiG-SCAPE installation plus a processed Pfam-A.hmm database

## Evidence

- [abstract] we found a putative reuterin biosynthetic gene cluster from a Streptococcus strain to be enriched but not exclusively found in healthy samples; metabolomic data from the same samples showed masses: "we found a putative reuterin biosynthetic gene cluster from a Streptococcus strain to be enriched but not exclusively found in healthy samples; metabolomic data from the same samples showed masses"
- [intro] Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] bigmap-download [Options]* -A [accession_list_file] -O [path_to_outdir]: [evidence span withheld — anchor status: external_doc]
- [readme] This script is created to easily download the metagenomic and/or metatranscriptomic samples available in the SRA repository: [evidence span withheld — anchor status: external_doc]
