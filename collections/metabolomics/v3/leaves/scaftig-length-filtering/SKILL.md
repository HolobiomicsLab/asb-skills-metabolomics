---
name: scaftig-length-filtering
description: Use when running MetaRon's 'ago' mode after IDBA assembly of paired-end
  shotgun reads, to discard scaftigs ≤ 500 bp prior to prodigal/MetaGeneMark gene
  prediction and proximon/operon identification.
when_to_use_negative:
- Input consists of pre-assembled metagenomic scaftigs and gene predictions supplied
  directly to MetaRon's 'op' process that have already been length-filtered upstream
- The analysis is whole-genome operon prediction on a complete reference genome (e.g.
  E. coli MG1655 NC_000913.3), where no de novo scaftig assembly occurs
- Scaftigs must be preserved regardless of length for a separate purpose (e.g. short-contig
  binning), since this filter permanently discards contigs ≤ 500 bp
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: MetaRon
  role: Pipeline that internally performs the ≤ 500 bp scaftig removal between IDBA
    assembly and gene prediction in 'ago' mode
  repo: https://github.com/zaidissa/MetaRon
- name: IDBA
  role: De Bruijn graph de novo assembler whose scaftig output is subjected to the
    length filter
- name: prodigal
  role: Gene predictor that runs on the length-filtered scaftigs to produce the .tab
    gene predictions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:991c0bbc3ed52bde33b8e87290b58fcbe8c2d7e44236d3b155f82e918615a945
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - prodigal
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# scaftig-length-filtering

## Summary

A quality-control filter step in the MetaRon metagenomic operon pipeline that discards de novo assembled scaftigs of length ≤ 500 bp before gene and operon prediction, keeping only sufficiently long contigs to support reliable gene calls.

## When to use

Use when running MetaRon's 'ago' mode after IDBA assembly of paired-end shotgun reads, to discard scaftigs ≤ 500 bp prior to prodigal/MetaGeneMark gene prediction and proximon/operon identification.

## When NOT to use

- Input consists of pre-assembled metagenomic scaftigs and gene predictions supplied directly to MetaRon's 'op' process that have already been length-filtered upstream
- The analysis is whole-genome operon prediction on a complete reference genome (e.g. E. coli MG1655 NC_000913.3), where no de novo scaftig assembly occurs
- Scaftigs must be preserved regardless of length for a separate purpose (e.g. short-contig binning), since this filter permanently discards contigs ≤ 500 bp

## Inputs

- Scaftigs in FASTA format produced by IDBA assembly of paired-end metagenomic shotgun reads (e.g. <sample>_scaf.fa)
- Per-sample paired-end FASTQ read files processed via 'metaron --process ago'

## Outputs

- Filtered scaftig FASTA containing only contigs longer than 500 bp
- Downstream gene-prediction input ( GenePrediction.tab from MetaGeneMark or Prodigal) built on the retained scaftigs

## How to apply

Assemble raw paired-end FASTQ reads into scaftigs de novo using IDBA as invoked by MetaRon's 'ago' process (e.g. in the E. coli K-12 MG1655 HiSeq run, MetaRon assembled 82 scaftigs). Remove every scaftig whose length is less than or equal to 500 bp, retaining only longer contigs for the downstream stages. This rationale is that very short scaftigs are unlikely to yield trustworthy gene predictions and co-directional proximons (genes clustered by IGD < 601 bp) and would inflate spurious operon counts in the final Operon_File.tab catalogue. The filtered scaftigs then feed gene prediction (--tool 1 for MetaGeneMark or --tool 2 for Prodigal) and subsequently NNPP2.2-based promoter prediction.

## Related tools

- **MetaRon** (Pipeline that internally performs the ≤ 500 bp scaftig removal between IDBA assembly and gene prediction in 'ago' mode) — https://github.com/zaidissa/MetaRon
- **IDBA** (De Bruijn graph de novo assembler whose scaftig output is subjected to the length filter)
- **prodigal** (Gene predictor that runs on the length-filtered scaftigs to produce the .tab gene predictions)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output ~/path/to/output/directory/
```

## Evaluation signals

- No scaftig of length ≤ 500 bp remains in the filtered FASTA file
- The 500 bp threshold is explicitly applied before, not after, gene prediction (4227 genes were predicted from the remaining scaftigs in the E. coli run)
- Count of scaftigs before and after filtering is documented; in the article's E. coli K-12 MG1655 HiSeq run, 82 scaftigs were assembled and the remainder passed the filter
- Final operon counts downstream (e.g. the 1,232,407 unique operons from 145 human gut samples) are reproducible only when the length filter was applied at the correct stage

## Limitations

- The 500 bp cutoff is a hard-coded pipeline choice; shorter scaftigs are discarded entirely and cannot contribute genes to operon predictions
- MetaRon operates on Linux and is restricted to academic use, so the filter must be reproduced manually on other platforms
- Python prerequisite is version 2.7 per the README, which may conflict with newer environments when re-implementing the filter

## Evidence

- [results] Scaftigs with length less than or equal to 500 bp were removed.: "Scaftigs with length less than or equal to 500 bp were removed."
- [results] 82 scaftigs assembled then genes predicted: "82 scaftigs were assembled by MetaRon via IDBA [55]."
- [readme] README description of the 'ago' pipeline stages: "assembly into scaffolds via IDBA, data manipulation, gene prediction via prodigal and lastly operon prediction based on gene's co-directionality, intergenic distance (IGD) and promoters."
