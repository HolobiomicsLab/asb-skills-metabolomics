---
name: metagenomic-read-mapping-abundance-profiling
description: Use when you have predicted MGC/BGC regions from antiSMASH/gutSMASH genomes
  plus shotgun metagenomic or metatranscriptomic reads (e.g., SRA accessions) and
  need RPKM abundance profiles and phenotype-differential cluster statistics.
when_to_use_negative:
- Input is already a gene cluster abundance/RPKM table and no read-to-cluster mapping
  is needed — use downstream statistics only
- The reads are targeted amplicon (16S) data rather than shotgun metagenomic or metatranscriptomic
  reads, so gene-cluster sequence mapping is not meaningful
- The gene clusters were not produced by antiSMASH or gutSMASH in a compatible output-folder
  layout, which bigmap-family requires as input
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: BiG-MAP
  role: 'Main pipeline: download, family grouping, mapping, and differential abundance
    analysis modules (bigmap-download/family/map/analyse)'
  repo: https://github.com/medema-group/BiG-MAP
- name: gutSMASH
  role: Predicts specialized primary metabolic gene clusters from reference genomes
    prior to abundance profiling
  repo: https://github.com/victoriapascal/gutsmash
- name: BiG-SCAPE
  role: Second-round redundancy filtering; groups gene clusters into GCFs
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: ncbi/sra-tools
  role: Underlying SRA access/fastq-dump used by bigmap-download to fetch and convert
    reads
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:a03449881303512eb1f501a658ea3b05071da845313b2ccf744237b2c4e1e74c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - bigmap-analyse
  - bigmap-map
  - gutSMASH
  - ncbi/sra-tools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 24
---

# metagenomic-read-mapping-abundance-profiling

## Summary

Profiles metabolic gene cluster (MGC) abundance in shotgun metagenomic samples by downloading SRA reads, redundantly filtering and grouping gene clusters (MinHash/BiG-SCAPE GCFs), mapping reads to representative cluster sequences to compute RPKM, and testing differential abundance (e.g., Kruskal-Wallis) between phenotypes.

## When to use

Use when you have predicted MGC/BGC regions from antiSMASH/gutSMASH genomes plus shotgun metagenomic or metatranscriptomic reads (e.g., SRA accessions) and need RPKM abundance profiles and phenotype-differential cluster statistics.

## When NOT to use

- Input is already a gene cluster abundance/RPKM table and no read-to-cluster mapping is needed — use downstream statistics only
- The reads are targeted amplicon (16S) data rather than shotgun metagenomic or metatranscriptomic reads, so gene-cluster sequence mapping is not meaningful
- The gene clusters were not produced by antiSMASH or gutSMASH in a compatible output-folder layout, which bigmap-family requires as input

## Inputs

- gutSMASH or antiSMASH output folders containing predicted gene clusters from reference genomes (e.g., 1,440 HMOD oral genomes yielding 3,352 MGCs)
- SRA accession list file (one run accession per line) for the shotgun metagenomic samples
- Processed Pfam-A.hmm database (hmmpress-processed) for BiG-SCAPE-based second-round filtering
- Sample phenotype/metadata table mapping accessions to conditions (e.g., healthy vs caries-associated)

## Outputs

- BiG-MAP.GCF.fna reference FASTA and BiG-MAP.GCF.bed core-region BED file for mapping
- BiG-MAP.GCs.json / BiG-MAP.GCF.json dictionaries of cluster families
- Per-sample RPKM abundance table of MGCs/GCFs across samples
- Differential abundance statistics per cluster (test statistic, P-value, group means, coverage metrics, e.g., saved to pdu_operon_verification.json)

## How to apply

Run gutSMASH (or antiSMASH) on reference genomes to predict MGCs, then feed the output folders into bigmap-family, which applies MinHash-based redundancy filtering (default 0.8 sequence-similarity cut-off) and optionally groups clusters into gene cluster families with BiG-SCAPE. Use bigmap-download with an SRA accession list file to fetch shotgun reads and convert them from .SRA to .fastq via fastq-dump. Run bigmap-map to align reads to the GCF reference FASTA (BiG-MAP.GCF.fna), extract core regions using the BED file, and compute per-sample RPKM abundance profiles; optionally filter by a core-coverage threshold (e.g., 0.5, as used in the 96-sample oral metagenome validation). Finally run bigmap-analyse to test differential abundance between conditions using a nonparametric test such as Kruskal-Wallis, and verify results against reported values (e.g., healthy 5.62 vs diseased 3.38 RPKM, P = 0.00049 for the Streptococcus sp. F0442 pdu operon).

## Related tools

- **BiG-MAP** (Main pipeline: download, family grouping, mapping, and differential abundance analysis modules (bigmap-download/family/map/analyse)) — https://github.com/medema-group/BiG-MAP
- **gutSMASH** (Predicts specialized primary metabolic gene clusters from reference genomes prior to abundance profiling) — https://github.com/victoriapascal/gutsmash
- **BiG-SCAPE** (Second-round redundancy filtering; groups gene clusters into GCFs) — https://github.com/medema-group/BiG-SCAPE/wiki
- **ncbi/sra-tools** (Underlying SRA access/fastq-dump used by bigmap-download to fetch and convert reads) — https://github.com/ncbi/sra-tools

## Examples

```
bigmap-download -A Acc_list.txt -O /usr001/fastq/schirmer/
```

## Evaluation signals

- bigmap-family produces all four expected artifacts: BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, BiG-MAP.GCF.json
- RPKM group means and Kruskal-Wallis P-values for a known cluster reproduce reported values (e.g., pdu operon: 5.62 vs 3.38 RPKM, P = 0.00049)
- Differential cluster counts match expectations (246 clusters whole set, 173 core in the oral validation)
- Coverage QC holds: mean core coverage 0.77 among healthy samples with coverage >= 0.5, and 13 of 34 healthy samples below the 0.5 coverage threshold are accounted for
- MinHash redundancy filtering at the default 0.8 similarity cut-off yields non-degenerate GCF counts (threshold tunable via -tg)

## Limitations

- BiG-MAP has been only tested on Ubuntu, per the repository README
- Ambiguous mapping between highly similar clusters is only avoided through redundancy filtering; the second BiG-SCAPE round is optional and its omission reduces filtering accuracy
- Metagenomic presence is partial: in the oral validation, 13 of 34 healthy samples fell below the 0.5 core-coverage threshold despite enrichment (P = 0.00049), so absence of coverage does not always equal absence of the cluster

## Evidence

- [intro] BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19): "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [intro] BiG-MAP.download, which uses a list of SRA accession numbers to download the shotgun data if present in the SRA database: "BiG-MAP.download, which uses a list of SRA accession numbers to download the shotgun data if present in the SRA database"
- [intro] Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests.: "Additionally, it calculates differential abundance or expression using either parametric or nonparametric tests."
- [readme] bigmap-download -A Acc_list.txt -O /usr001/fastq/schirmer/: [evidence span withheld — anchor status: external_doc]
- [readme] The first redundancy filtering step is performed by MASH, that by default uses 0.8 sequence similarity cut-off but can be changed as desired.: [evidence span withheld — anchor status: external_doc]
