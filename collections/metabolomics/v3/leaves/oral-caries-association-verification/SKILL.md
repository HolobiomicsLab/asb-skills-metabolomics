---
name: oral-caries-association-verification
description: Use when reproducing or auditing the reported differential abundance
  of the S. mutans 14D muc operon across 96 healthy vs caries oral metagenomes, or
  validating BiG-MAP fitZIG output for a specific gene cluster against published coverage
  statistics.
when_to_use_negative:
- When the analysis target is the smaller Aleti et al. (PRJNA478018) subset alone
  — the paper reports no significant muc operon difference there, so a positive-control
  verification is not expected to reproduce
- When input gene clusters have not been redundancy-filtered/GCF-grouped (raw antiSMASH
  output of 1,849 BGCs) — mapping without MinHash/BiG-SCAPE filtering causes ambiguous
  mapping
- When the goal is general-purpose metagenome functional profiling with reference
  databases (KEGG/MetaCyc/HUMAnN2) rather than BGC/GCF-level abundance profiling
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BiG-MAP
  role: Four-module pipeline (bigmap-download, bigmap-family, bigmap-map, bigmap-analyse)
    used to download SRA data, group BGCs into GCFs, map metagenomic reads, and run
    differential abundance testing
  repo: github.com/medema-group/BiG-MAP
- name: antiSMASH
  role: Mines the 170 S. mutans genomes for biosynthetic gene clusters feeding bigmap-family
- name: BiG-SCAPE
  role: Second redundancy-filtering round; groups BGCs into gene cluster families
    (GCFs) within bigmap-family
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: sra-tools
  role: Provides fastq-dump used by bigmap-download to convert downloaded .SRA files
    to .fastq
  repo: github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:a3ee2fea827de268d34ea70bd94bd03b775096e715b88443c568dda551ce9c1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - antiSMASH
  - bigmap-analyse
  - bigmap-map
  - sra-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# oral-caries-association-verification

## Summary

Verifies that the S. mutans muc operon (PKS/NRPS tetramic-acid BGC) is differentially abundant between healthy and caries-associated oral metagenomes by reproducing the BiG-MAP profiling workflow (download, antiSMASH mining, GCF grouping, mapping, fitZIG differential testing).

## When to use

Use when reproducing or auditing the reported differential abundance of the S. mutans 14D muc operon across 96 healthy vs caries oral metagenomes, or validating BiG-MAP fitZIG output for a specific gene cluster against published coverage statistics.

## When NOT to use

- When the analysis target is the smaller Aleti et al. (PRJNA478018) subset alone — the paper reports no significant muc operon difference there, so a positive-control verification is not expected to reproduce
- When input gene clusters have not been redundancy-filtered/GCF-grouped (raw antiSMASH output of 1,849 BGCs) — mapping without MinHash/BiG-SCAPE filtering causes ambiguous mapping
- When the goal is general-purpose metagenome functional profiling with reference databases (KEGG/MetaCyc/HUMAnN2) rather than BGC/GCF-level abundance profiling

## Inputs

- SRA accession list file for the 96 healthy/caries oral metagenome samples (e.g. from BioProject PRJNA478018 and related cohorts)
- 170 S. mutans genome assemblies (Tang et al. and Liu et al. cohorts)
- antiSMASH output folders for the mined S. mutans genomes
- Processed Pfam-A.hmm database for BiG-SCAPE (hmmpress-processed)
- Phenotype metadata labeling samples as healthy vs caries-associated

## Outputs

- BiG-MAP.GCF.fna / BiG-MAP.GCF.bed / BiG-MAP.GCs.json / BiG-MAP.GCF.json from bigmap-family (GCF reference and grouping files)
- Per-sample core-coverage abundance profiles per GCF from bigmap-map
- fitZIG differential abundance results for healthy vs caries phenotypes
- differential_abundance_results.tsv containing the differential abundance table and muc operon verification summary

## How to apply

Download the 96 oral metagenome samples from SRA with bigmap-download using an accession list (-A), and mine the 170 S. mutans genomes (Tang et al. and Liu et al. cohorts) with antiSMASH to obtain 1,849 predicted BGCs. Reduce these to 41 nonredundant representatives with bigmap-family (MASH redundancy filtering with 0.8 similarity cut-off, plus BiG-SCAPE GCF grouping). Map the metagenomes to representative gene cluster sequences with bigmap-map to obtain per-sample core-coverage abundance profiles for each GCF. Run differential abundance between healthy and caries phenotypes with bigmap-analyse using the fitZIG zero-inflated model (not the Kruskal-Wallis nonparametric test). Verify the muc operon against the reported values: mean core coverage 0.274 (healthy) vs 0.161 (caries); after filtering to coverage >=0.5 (8 healthy, 6 disease samples), filtered means of 0.848 and 0.997 with minimal fold change between group averages; and no significant difference in the smaller Aleti et al. (PRJNA478018) subset. Write the differential abundance table and muc operon verification summary to differential_abundance_results.tsv.

## Related tools

- **BiG-MAP** (Four-module pipeline (bigmap-download, bigmap-family, bigmap-map, bigmap-analyse) used to download SRA data, group BGCs into GCFs, map metagenomic reads, and run differential abundance testing) — github.com/medema-group/BiG-MAP
- **antiSMASH** (Mines the 170 S. mutans genomes for biosynthetic gene clusters feeding bigmap-family)
- **BiG-SCAPE** (Second redundancy-filtering round; groups BGCs into gene cluster families (GCFs) within bigmap-family) — https://github.com/medema-group/BiG-SCAPE/wiki
- **sra-tools** (Provides fastq-dump used by bigmap-download to convert downloaded .SRA files to .fastq) — github.com/ncbi/sra-tools

## Examples

```
bigmap-download -A Acc_list.txt -O /usr001/fastq/caries/ && bigmap-family -D /usr001/mutans_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- fitZIG (not Kruskal-Wallis) flags the muc operon as differentially abundant between healthy and caries phenotypes
- Mean core coverage of the muc operon approximates 0.274 (healthy) and 0.161 (caries) across all samples
- Coverage >=0.5 filtering retains 8 healthy and 6 disease samples with filtered mean coverages near 0.848 and 0.997, and minimal fold change between group averages
- antiSMASH mining of 170 genomes yields ~1,849 BGCs, reduced to 41 nonredundant representatives after MinHash/BiG-SCAPE filtering
- The Aleti et al. (PRJNA478018) subset shows no significant muc operon difference, matching the published negative control

## Limitations

- BiG-MAP has been only tested on Ubuntu; other platforms are unsupported per the README
- Differential abundance conclusions depend on the statistical model choice — the muc operon is fitZIG-significant but the result differs under Kruskal-Wallis
- Low-coverage samples confound group means; a coverage >=0.5 filter (leaving 8 healthy, 6 disease samples) is needed before comparing filtered mean coverages, and the small Aleti et al. subset lacks power to detect a difference

## Evidence

- [abstract] In 96 metagenomic samples from healthy and caries-associated oral microbiomes, 252 gene clusters were significantly more abundant in either phenotype.: "we analyzed 96 metagenomic samples from healthy and caries-associated human oral microbiome samples and identified 252 gene clusters, including unreported ones, that were significantly more abundant"
- [abstract] The muc operon was among the differentially abundant gene clusters.: "Among them, we found the muc operon, a gene cluster known to be associated with tooth decay."
- [intro] BiG-MAP uses MinHash-based redundancy filtering and BiG-SCAPE GCF grouping.: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [intro] Differential abundance is calculated with parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] bigmap-family performs MASH redundancy filtering with a default 0.8 similarity cut-off.: [evidence span withheld — anchor status: external_doc]
- [readme] BiG-MAP is composed of four modules covering download, family grouping, mapping, and analysis.: "bigmap/download.py
- bigmap/family.py
- bigmap/map.py
- bigmap/analyse.py"
