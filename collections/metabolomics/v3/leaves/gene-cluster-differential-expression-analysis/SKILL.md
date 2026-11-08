---
name: gene-cluster-differential-expression-analysis
description: Use when you have antiSMASH or gutSMASH cluster predictions from a reference
  genome collection and paired shotgun metagenomic/metatranscriptomic reads, and need
  to test which gene clusters differ in abundance or expression between phenotypes
  (e.g. Crohn's disease vs healthy).
when_to_use_negative:
- The input is already a gene cluster abundance/expression feature table — only the
  bigmap-analyse step applies, not the full map-and-test workflow.
- 'No antiSMASH/gutSMASH cluster predictions exist for the reference genomes: bigmap-family
  requires pre-computed anti-/gutSMASH output folders as input and cannot start from
  raw genomes.'
- The data are 16S amplicon or other non-shotgun reads, since the method relies on
  mapping reads to gene cluster nucleotide sequences.
- The working environment is not Ubuntu-based, as the README states the tool 'has
  been only tested on Ubuntu'.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3697
tools:
- name: BiG-MAP
  role: 'Full pipeline: bigmap-download fetches SRA reads, bigmap-family builds GCFs,
    bigmap-map quantifies abundance/expression, bigmap-analyse runs the differential
    tests'
  repo: github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Second-round redundancy filtering of gene clusters into GCFs during bigmap-family
- name: gutSMASH
  role: Predicts specialized primary metabolic gene clusters (MGCs) from the reference
    genomes that become the mapping targets
  repo: victoriapascal/gutsmash
- name: sra-tools
  role: Underlying SRA access/fastq-dump conversion used by bigmap-download
  repo: ncbi/sra-tools
- name: seaborn
  role: Statistical visualization of abundance/expression profile outputs
  repo: mwaskom/seaborn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:4289d926c08bbf4d260b1ec70e1c2aab690835a042ee2428a2397e1d36d3002e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - gutSMASH
  - seaborn
  - sra-tools
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gene-cluster-differential-expression-analysis

## Summary

Profile metabolic gene cluster (MGC) and gene cluster family (GCF) abundance and expression across metagenomic and metatranscriptomic samples by mapping reads to representative cluster sequences, then call differentially abundant/expressed clusters between conditions using nonparametric tests. This is the core analysis capability of the BiG-MAP pipeline.

## When to use

Use when you have antiSMASH or gutSMASH cluster predictions from a reference genome collection and paired shotgun metagenomic/metatranscriptomic reads, and need to test which gene clusters differ in abundance or expression between phenotypes (e.g. Crohn's disease vs healthy).

## When NOT to use

- The input is already a gene cluster abundance/expression feature table — only the bigmap-analyse step applies, not the full map-and-test workflow.
- No antiSMASH/gutSMASH cluster predictions exist for the reference genomes: bigmap-family requires pre-computed anti-/gutSMASH output folders as input and cannot start from raw genomes.
- The data are 16S amplicon or other non-shotgun reads, since the method relies on mapping reads to gene cluster nucleotide sequences.
- The working environment is not Ubuntu-based, as the README states the tool 'has been only tested on Ubuntu'.

## Inputs

- antiSMASH or gutSMASH output folders (predicted gene clusters per reference genome)
- SRA accession list file (e.g. from BioProject PRJNA389280)
- Shotgun metagenomic and metatranscriptomic FASTQ reads
- Processed Pfam-A.hmm database (uncompressed and hmmpress-processed) for BiG-SCAPE redundancy filtering

## Outputs

- BiG-MAP.GCF.fna, BiG-MAP.GCF.bed, BiG-MAP.GCs.json, BiG-MAP.GCF.json (representative cluster references and family dictionaries)
- Per-sample MGC/GCF abundance and expression profile tables
- Differential abundance/expression results table (e.g. bigmap_cd_differential_results.tsv) with Kruskal-Wallis test calls per cluster
- Visualization of abundance/expression profiles across samples

## How to apply

Download the metagenomic/metatranscriptomic FASTQ files with bigmap-download using an SRA accession list (e.g. BioProject PRJNA389280 from the Schirmer et al. IBD cohort). Run bigmap-family on the gutSMASH output folders to first filter redundant clusters with MASH at the default 0.8 similarity threshold, optionally (recommended) adding a second BiG-SCAPE redundancy-filtering round that requires a hmmpress-processed Pfam-A.hmm database; this produces BiG-MAP.GCF.fna, BiG-MAP.GCF.bed, and the GCF dictionaries. Run bigmap-map to map the WGS reads to the representative GCF sequences and quantify per-sample MGC/GCF abundance and expression profiles. Finally run bigmap-analyse, choosing the nonparametric Kruskal-Wallis test to compare conditions (e.g. CD vs healthy) and to compute differential abundance and differential expression. Correctness is judged by recovering the reported result scale — in the Schirmer IBD cohort, 11 clusters significantly differentially abundant and 8 significantly differentially expressed — and writing the results table (e.g. bigmap_cd_differential_results.tsv).

## Related tools

- **BiG-MAP** (Full pipeline: bigmap-download fetches SRA reads, bigmap-family builds GCFs, bigmap-map quantifies abundance/expression, bigmap-analyse runs the differential tests) — github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Second-round redundancy filtering of gene clusters into GCFs during bigmap-family)
- **gutSMASH** (Predicts specialized primary metabolic gene clusters (MGCs) from the reference genomes that become the mapping targets) — victoriapascal/gutsmash
- **sra-tools** (Underlying SRA access/fastq-dump conversion used by bigmap-download) — ncbi/sra-tools
- **seaborn** (Statistical visualization of abundance/expression profile outputs) — mwaskom/seaborn

## Examples

```
bigmap-download -A Acc_list.txt -O /usr001/fastq/schirmer/ && bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- bigmap-family produces all four expected artifacts: BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json.
- In the PRJNA389280 IBD cohort run, the Kruskal-Wallis results reproduce 11 significantly differentially abundant (metagenomic) and 8 significantly differentially expressed (metatranscriptomic) gene clusters between CD and healthy samples.
- The differential results are written to a TSV (e.g. bigmap_cd_differential_results.tsv) with one call per MGC/GCF.
- Redundancy-filtering settings are honored: MASH similarity threshold defaults to 0.8 (-tg, fraction 0–1), and housekeeping gene similarity threshold defaults to 0.1 (-th).
- Per-sample abundance/expression profiles are non-ambiguous, confirming the MinHash-based redundancy filtering removed clusters that would map ambiguously.

## Limitations

- The tool has been only tested on Ubuntu; other platforms are unsupported.
- The BiG-SCAPE second redundancy-filtering step is optional and requires a separate conda install plus a hmmpress-processed Pfam-A.hmm database; skipping it leaves only the MASH 0.8 similarity filtering.
- MinHash redundancy filtering at the default 0.8 threshold may need tuning via -tg (0 = every cluster is its own family, 1 = one large family), affecting downstream GCF-level calls.
- Only a single nonparametric (Kruskal-Wallis) or parametric test choice is available within bigmap-analyse; very small per-group sample sizes (the tutorial uses 8 samples) limit statistical power.

## Evidence

- [intro] MinHash-based redundancy filtering and GCF grouping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [intro] Differential testing options: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [abstract] Tool purpose statement: [evidence span withheld — anchor status: external_doc]
- [readme] bigmap-analyse role in workflow: [evidence span withheld — anchor status: external_doc]
- [readme] Default MASH similarity threshold: [evidence span withheld — anchor status: external_doc]
