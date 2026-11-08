---
name: metatranscriptomic-expression-profiling
description: Use when metatranscriptomic (RNA-seq) reads from a microbiome cohort
  must be mapped against gutSMASH/antiSMASH-predicted gene cluster sequences to obtain
  per-sample MGC/GCF expression profiles and detect clusters differentially expressed
  between conditions (e.g. Crohn's disease vs healthy).
when_to_use_negative:
- Only metagenomic (DNA) data are available and no metatranscriptomic reads exist
  — expression profiling is not possible; abundance profiling alone applies
- The goal is pathway/function abundance based on reference databases like KEGG or
  MetaCyc (tools such as HUMAnN2, MetaPath, FMAP, Metatrans are the wrong choice);
  BiG-MAP does not depend on such pathway databases
- Gene clusters have not yet been predicted by gutSMASH/antiSMASH for the reference
  genomes — bigmap-family and bigmap-map have no input
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BiG-MAP
  role: 'Main pipeline: bigmap-family (GCF grouping/redundancy filtering), bigmap-map
    (read mapping and expression quantification), bigmap-analyse (differential expression
    testing)'
  repo: github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Second redundancy-filtering round that groups BGCs into GCFs more accurately
    before mapping metatranscriptomic reads
- name: gutSMASH
  role: Predicts the specialized-primary-metabolism gene clusters from reference genomes
    that serve as the mapping reference for expression profiling
  repo: github.com/victoriapascal/gutsmash
- name: seaborn
  role: Statistical visualization of expression profiles across samples
  repo: github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:daf68ef7a7a2af0f707b6929fff1d17833b0d895a72c5035886dc4c2e9870f22
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - gutSMASH
  - seaborn
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# metatranscriptomic-expression-profiling

## Summary

Profiles metabolic gene cluster (MGC) expression by mapping metatranscriptomic reads to gene cluster sequences from reference genomes (e.g. gutSMASH predictions), then testing differential expression between conditions with parametric or nonparametric statistics via BiG-MAP.

## When to use

Use when metatranscriptomic (RNA-seq) reads from a microbiome cohort must be mapped against gutSMASH/antiSMASH-predicted gene cluster sequences to obtain per-sample MGC/GCF expression profiles and detect clusters differentially expressed between conditions (e.g. Crohn's disease vs healthy).

## When NOT to use

- Only metagenomic (DNA) data are available and no metatranscriptomic reads exist — expression profiling is not possible; abundance profiling alone applies
- The goal is pathway/function abundance based on reference databases like KEGG or MetaCyc (tools such as HUMAnN2, MetaPath, FMAP, Metatrans are the wrong choice); BiG-MAP does not depend on such pathway databases
- Gene clusters have not yet been predicted by gutSMASH/antiSMASH for the reference genomes — bigmap-family and bigmap-map have no input

## Inputs

- Metatranscriptomic FASTQ reads from shotgun metatranscriptomic samples (e.g. SRA accessions from BioProject PRJNA389280)
- gutSMASH (or antiSMASH) output directories containing predicted gene clusters for the reference genomes
- Processed Pfam-A.hmm database (hmmpress'ed) for BiG-SCAPE
- SRA accession list file for the cohort of interest

## Outputs

- Per-sample MGC/GCF expression profile matrix (bigmap-map output)
- BiG-MAP.GCF.fna reference file, BiG-MAP.GCF.bed core-region bedfile, BiG-MAP.GCs.json and BiG-MAP.GCF.json GCF dictionaries
- Differential expression statistics table (e.g. bigmap_cd_differential_results.tsv with Kruskal-Wallis results)
- Expression profile visualizations across samples

## How to apply

First group predicted gene clusters into gene cluster families (GCFs) with bigmap-family, using MinHash (MASH) redundancy filtering with the default 0.8 similarity cut-off and a recommended second BiG-SCAPE filtering round (with a processed, hmmpress'ed Pfam-A database) to avoid ambiguous mapping. Then run bigmap-map to align the metatranscriptomic reads to the representative gene cluster sequences in BiG-MAP.GCF.fna (using BiG-MAP.GCF.bed to extract core regions) and quantify per-sample MGC/GCF expression profiles; housekeeping gene family (HGF) representatives are assessed in parallel for normalization context. Finally run bigmap-analyse to compute differential expression with parametric or nonparametric tests — the IBD case study uses the Kruskal-Wallis test on Crohn's disease vs healthy metatranscriptomes. A correct application on the Schirmer et al. (2018) cohort (PRJNA389280) recovers 8 significantly differentially expressed gene clusters; expression values should be output and visualized as profiles across samples (e.g. with seaborn plots).

## Related tools

- **BiG-MAP** (Main pipeline: bigmap-family (GCF grouping/redundancy filtering), bigmap-map (read mapping and expression quantification), bigmap-analyse (differential expression testing)) — github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Second redundancy-filtering round that groups BGCs into GCFs more accurately before mapping metatranscriptomic reads)
- **gutSMASH** (Predicts the specialized-primary-metabolism gene clusters from reference genomes that serve as the mapping reference for expression profiling) — github.com/victoriapascal/gutsmash
- **seaborn** (Statistical visualization of expression profiles across samples) — github.com/mwaskom/seaborn

## Examples

```
bigmap-download -A Acc_list.txt -O /usr001/fastq/schirmer/ ; bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- bigmap-map produces a per-sample expression matrix keyed to GCF representatives, consistent with the BiG-MAP.GCF.fna and BiG-MAP.GCs.json/BiG-MAP.GCF.json dictionaries from bigmap-family
- On the Schirmer et al. IBD cohort (PRJNA389280), Kruskal-Wallis analysis of metatranscriptomes identifies 8 significantly differentially expressed gene clusters between Crohn's disease and healthy samples
- Redundancy-filtering settings are verified: MASH cut-off at the default 0.8 similarity and BiG-SCAPE GCF grouping completed before mapping
- Expression profiles are output and visualized across all samples, alongside housekeeping gene family (HGF) representative profiles
- No ambiguous mapping artefacts: gene clusters with high sequence redundancy were collapsed into families rather than double-counted

## Limitations

- BiG-MAP has only been tested on Ubuntu
- Requires prior gutSMASH/antiSMASH runs for the reference genomes; the modules cannot predict clusters themselves
- Ambiguous mapping is a key failure mode — skipping the MinHash redundancy filtering (0.8 cut-off) or the recommended BiG-SCAPE second filtering round can inflate or distort expression estimates
- Results depend on reference genome choice: the IBD study used gutSMASH predictions from the 1,520 CGR genome collection as reference

## Evidence

- [abstract] profiles abundance and expression levels of gene clusters across metagenomic and metatranscriptomic data and evaluates differential abundance/expression under different conditions.: "BiG-MAP, a bioinformatic tool to profile abundance and expression levels of gene clusters across metagenomic and metatranscriptomic data and evaluate their differential abundance and expression under"
- [intro] Reads are mapped to gene cluster sequences and differential tests are run: "by mapping metagenomic and metatranscriptomic reads to gene cluster sequences from reference genomes or metagenomic assemblies"
- [intro] Differential abundance or expression is calculated using parametric or nonparametric tests: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] bigmap-map assesses expression profiles; bigmap-analyse performs differential analysis: "3) Assess gene cluster (and HGF representatives) abundance and expression profiles
   using bigmap-map
4) Performs differential abundance/expression analysis and visualizes the output
   using"
- [readme] MASH redundancy filtering uses a default 0.8 similarity cut-off; BiG-SCAPE round recommended: [evidence span withheld — anchor status: external_doc]
- [readme] The IBD tutorial cohort is the Schirmer et al. (2018) PRJNA389280 BioProject: [evidence span withheld — anchor status: external_doc]
