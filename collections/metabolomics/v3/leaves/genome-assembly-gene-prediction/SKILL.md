---
name: genome-assembly-gene-prediction
description: Use when starting from filtered raw metagenomic or whole-genome shotgun
  reads (FASTQ) and you need assembled scaftigs plus gene predictions as input for
  operon or promoter-level prediction.
when_to_use_negative:
- Input is already an assembled genome or scaftigs with gene calls — use MetaRon 'op'
  mode instead of re-assembling
- Operon prediction must rely on functional/experimental annotation rather than de
  novo co-directionality, IGD and promoter signals — MetaRon is designed to work without
  functional information, but other annotation-driven tools may be more appropriate
- Data are single-end reads or reads that the pipeline's paired-end read_type options
  ('merge'/'paired') cannot represent
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaRon
  role: Pipeline that orchestrates de novo assembly, gene prediction and downstream
    operon prediction; exposes --process ago and --process op modes
  repo: https://github.com/zaidissa/MetaRon
- name: IDBA
  role: Iterative De Bruijn Graph de novo assembler used inside MetaRon to assemble
    raw reads into scaftigs (82 scaftigs in the E. coli K-12 MG1655 run)
- name: prodigal
  role: Gene caller applied to the assembled scaftigs (MetaRon --tool 2); predicted
    4227 genes on the E. coli K-12 MG1655 scaftigs
- name: NNPP
  role: Neural Network Promoter Prediction 2.2, whose path must be registered in config.txt
    so de novo promoter evidence is incorporated after gene prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:aa7d6eb17d4aab67b0445a59923275ade4233ad6a5c031dc1abdf1ff50022b8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - prodigal
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# genome-assembly-gene-prediction

## Summary

Assemble filtered shotgun reads into scaftigs with IDBA and predict genes with Prodigal (or MetaGeneMark), producing the scaftig FASTA and gene-prediction .tab files that MetaRon requires for downstream operon prediction.

## When to use

Use when starting from filtered raw metagenomic or whole-genome shotgun reads (FASTQ) and you need assembled scaftigs plus gene predictions as input for operon or promoter-level prediction.

## When NOT to use

- Input is already an assembled genome or scaftigs with gene calls — use MetaRon 'op' mode instead of re-assembling
- Operon prediction must rely on functional/experimental annotation rather than de novo co-directionality, IGD and promoter signals — MetaRon is designed to work without functional information, but other annotation-driven tools may be more appropriate
- Data are single-end reads or reads that the pipeline's paired-end read_type options ('merge'/'paired') cannot represent

## Inputs

- Filtered paired-end raw reads in FASTQ (two files for 'merge' read_type, one interleaved file for 'paired')
- Optional pre-assembled scaftig FASTA (e.g. ERR022075_scaf.fa) when using operon-prediction-only mode
- Optional gene prediction .tab file from Prodigal or MetaGeneMark

## Outputs

- Assembled scaftigs (FASTA) from IDBA, with scaftigs <= 500 bp removed
- Gene prediction .tab file (e.g. 4227 genes predicted with Prodigal in the E. coli K-12 MG1655 run)
- Scaftig set and gene predictions feeding MetaRon's co-directional proximon and operon prediction

## How to apply

Run MetaRon in 'ago' (assembly, gene prediction and operon prediction) mode by supplying paired-end reads via --paired_1/--paired_2 (read_type 'merge' for two files, 'paired' for one file) and a read_length flag ('l' if reads are longer than 128 bases, 'r' if shorter). The pipeline assembles reads into scaftigs via IDBA, removes scaftigs with length less than or equal to 500 bp, and predicts genes on the remaining scaftigs with Prodigal (--tool 2; MetaGeneMark via --tool 1 is the alternative). If scaftigs and gene predictions already exist, skip assembly and run MetaRon in 'op' mode, passing the Prodigal .tab file via --igp and the scaftig FASTA via --isc. Rationale: gene prediction on scaftigs (rather than raw reads) provides the co-directional gene pairs and intergenic distances that operon prediction depends on.

## Related tools

- **MetaRon** (Pipeline that orchestrates de novo assembly, gene prediction and downstream operon prediction; exposes --process ago and --process op modes) — https://github.com/zaidissa/MetaRon
- **IDBA** (Iterative De Bruijn Graph de novo assembler used inside MetaRon to assemble raw reads into scaftigs (82 scaftigs in the E. coli K-12 MG1655 run))
- **prodigal** (Gene caller applied to the assembled scaftigs (MetaRon --tool 2); predicted 4227 genes on the E. coli K-12 MG1655 scaftigs)
- **NNPP** (Neural Network Promoter Prediction 2.2, whose path must be registered in config.txt so de novo promoter evidence is incorporated after gene prediction)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ERR022075.1.fastq --paired_2 ERR022075.2.fastq --output ~/out/
```

## Evaluation signals

- Assembly produces scaftigs and scaftigs <= 500 bp are removed before gene prediction, matching the article's filter step
- Gene prediction yields a Prodigal-format .tab file with gene coordinates on the scaftigs; the reference run predicted 4227 genes from the remaining scaftigs of E. coli K-12 MG1655
- MetaRon exits cleanly on 'metaron --help' showing the 'ago' and 'op' process options, and config.txt contains the NNPP2.2 path
- Downstream operon prediction (Operon_File.tab) is produced with IGD threshold <= 600 bp and co-directionality criteria applied to the predicted genes
- On the E. coli C20 chicken-gut draft genome benchmark, the downstream operon predictions reproduce sensitivity, specificity and accuracy of 87, 91 and 88%

## Limitations

- Pipeline restricted to Linux and academic use only
- Python 2.7 is required per the README prerequisites (article discussion mentions Python > 3.0 — a version ambiguity to check before installing)
- Scaftigs <= 500 bp are discarded, so genes on very short contigs are lost
- NNPP's inability to identify some promoters limits downstream operon calls that depend on promoter evidence

## Evidence

- [results] 82 scaftigs were assembled by MetaRon via IDBA [55].: "82 scaftigs were assembled by MetaRon via IDBA [55]."
- [results] Scaftigs <= 500 bp removed: "Scaftigs with length less than or equal to 500 bp were removed."
- [results] 4227 genes predicted using prodigal: "The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32]."
- [intro] downstream data processing steps: "performs necessary downstream data processing (de novo assembly, gene prediction, de novo promoter prediction and proximon prediction), before identifying the operons"
- [readme] operon-only mode invocation from README: "metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 1 OR 2 --output ~/path/to/output/directory/"
- [readme] read length flag semantics from README: "Enter 'l' if read length is longer than 128 bases and 'r' if read length is shorter than 128 bases"
