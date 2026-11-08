---
name: metagenomic-gene-prediction
description: Use when shotgun metagenomic reads or scaftigs (FASTA) lack gene annotations
  and gene coordinates are required as input for operon/proximon prediction (e.g.
  MetaRon 'ago' or 'op' mode).
when_to_use_negative:
- The input is already a gene annotation (.tab) from MetaGeneMark or Prodigal and
  you only need operon prediction — use MetaRon 'op' mode directly instead of re-running
  assembly and gene prediction
- The goal is functional/pathway annotation or secondary metabolite analysis of genes
  — MetaRon's analysis of secondary metabolites and pathways was performed separately
  (antiSMASH/BOWTIE/FMAP), not integrated into gene prediction
- Data are eukaryotic or transcriptomic; the pipeline is restricted to bacterial shotgun
  metagenomic/whole-genome data on Linux for academic use
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaRon
  role: Pipeline orchestrating assembly, gene prediction (--tool 1 MetaGeneMark or
    --tool 2 Prodigal) and operon prediction
  repo: https://github.com/zaidissa/MetaRon
- name: prodigal
  role: Predicts genes on IDBA-assembled scaftigs (e.g. 4227 genes from E. coli K-12
    MG1655 HiSeq scaftigs)
- name: IDBA
  role: De novo assembles raw metagenomic reads into scaftigs prior to gene prediction
- name: NNPP
  role: De novo promoter prediction used downstream to define operon boundaries within
    proximon gene clusters
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:4fb57276b200dfd63ca971b4029abaa6a05a9e6b55d862f071c613f7668fad07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - NeSSM
  - prodigal
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenomic-gene-prediction

## Summary

Predicts protein-coding genes on de novo-assembled metagenomic scaftigs using Prodigal (or MetaGeneMark) as implemented in MetaRon, providing the gene coordinates needed for downstream proximon and operon prediction without functional or experimental information.

## When to use

Use when shotgun metagenomic reads or scaftigs (FASTA) lack gene annotations and gene coordinates are required as input for operon/proximon prediction (e.g. MetaRon 'ago' or 'op' mode).

## When NOT to use

- The input is already a gene annotation (.tab) from MetaGeneMark or Prodigal and you only need operon prediction — use MetaRon 'op' mode directly instead of re-running assembly and gene prediction
- The goal is functional/pathway annotation or secondary metabolite analysis of genes — MetaRon's analysis of secondary metabolites and pathways was performed separately (antiSMASH/BOWTIE/FMAP), not integrated into gene prediction
- Data are eukaryotic or transcriptomic; the pipeline is restricted to bacterial shotgun metagenomic/whole-genome data on Linux for academic use

## Inputs

- De novo assembled scaftigs FASTA (from IDBA, with scaftigs <= 500 bp removed)
- Paired-end shotgun metagenomic reads (FASTQ, two files) for the 'ago' end-to-end process
- Pre-computed gene prediction .tab file (MetaGeneMark or Prodigal format) for 'op' mode

## Outputs

- Gene prediction .tab file with gene coordinates on scaftigs
- Co-directional proximon clusters (IGD < 601 bp)
- Tab-delimited Operon_File.tab catalogue of predicted operons

## How to apply

In MetaRon's 'ago' process, raw paired-end FASTQ reads are first de novo assembled into scaftigs with IDBA, scaftigs of length <= 500 bp are removed, and the remaining scaftigs are scanned for genes with Prodigal (--tool 2) or MetaGeneMark (--tool 1). If pre-assembled scaftigs and gene predictions already exist, skip assembly and supply the gene prediction .tab file via --igp and the scaftig FASTA via --isc under process 'op'. The gene predictions feed the co-directional proximal gene cluster (proximon) stage, which requires intergenic distance (IGD) < 601 bp, and subsequently promoter-based operon boundary definition with NNPP2.2. Output is written as tab-delimited gene/operon records culminating in Operon_File.tab in the specified output directory.

## Related tools

- **MetaRon** (Pipeline orchestrating assembly, gene prediction (--tool 1 MetaGeneMark or --tool 2 Prodigal) and operon prediction) — https://github.com/zaidissa/MetaRon
- **prodigal** (Predicts genes on IDBA-assembled scaftigs (e.g. 4227 genes from E. coli K-12 MG1655 HiSeq scaftigs))
- **IDBA** (De novo assembles raw metagenomic reads into scaftigs prior to gene prediction)
- **NNPP** (De novo promoter prediction used downstream to define operon boundaries within proximon gene clusters)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output ~/path/to/output/directory/
```

## Evaluation signals

- Scaftigs shorter than or equal to 500 bp must be filtered out before gene prediction
- On E. coli K-12 MG1655 HiSeq reads (SRP029211), the reference run assembled 82 scaftigs and predicted 4227 genes with prodigal — a sanity-check scale expectation
- Gene predictions must be in the .tab format documented for MetaRon's --igp parameter (MetaGeneMark, tool 1, or Prodigal, tool 2)
- Downstream proximons must contain only co-directional gene pairs with IGD < 601 bp
- Final Operon_File.tab must exist, be tab-delimited, and conform to the scaftig/gene-prediction file format required by the 'op' process

## Limitations

- Requires the NNPP2.2 executable path to be added to config.txt for promoter-based boundary refinement; promoter prediction can fail on some genes ('the inability of the NNPP tool to identify the promoter')
- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated into the pipeline (planned future work)
- Pipeline is restricted to Linux and academic use only

## Evidence

- [results] The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32].: "The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32]."
- [results] Scaftigs with length less than or equal to 500 bp were removed.: "Scaftigs with length less than or equal to 500 bp were removed."
- [readme] Select the gene prediction .tab file generated via MetageneMark or Prodigal: "Select the gene prediction .tab file generated via MetageneMark or Prodigal"
- [intro] performs necessary downstream data processing (de novo assembly, gene prediction, de novo promoter prediction and proximon prediction), before identifying the operons: "performs necessary downstream data processing (de novo assembly, gene prediction, de novo promoter prediction and proximon prediction), before identifying the operons"
- [results] MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes.: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
