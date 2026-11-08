---
name: operon-prediction-pipeline-execution
description: Use when input is filtered shotgun FASTQ (or pre-assembled scaftigs plus
  a Prodigal/MetaGeneMark gene-prediction .tab) from a bacterial genome or metagenome
  and de novo operon predictions are the target output.
when_to_use_negative:
- The goal is functional/KEGG annotation or secondary-metabolite analysis rather than
  operon structure — the article notes secondary metabolite prediction and pathway
  annotation are not yet integrated into the pipeline and were performed separately
- The environment is not Linux — the README states the pipeline is restricted to Linux
  and academic use
- A curated reference genome operon annotation already exists and de novo metagenomic
  prediction is unnecessary
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaRon
  role: Master pipeline orchestrating assembly, gene prediction, promoter prediction,
    proximon identification, and operon prediction; installed via pip install metaron
  repo: github.com/zaidissa/MetaRon
- name: IDBA
  role: De novo assembles filtered shotgun reads into scaftigs inside the 'ago' process
  repo: github.com/zaidissa/MetaRon
- name: prodigal
  role: Predicts genes on filtered scaftigs (selected with --tool 2 in 'op' mode);
    alternative gene caller is MetaGeneMark (--tool 1)
  repo: github.com/zaidissa/MetaRon
- name: NNPP
  role: Performs de novo promoter prediction (BDGP Neural Network Promoter Prediction
    2.2); its executable path must be registered in config.txt
- name: NeSSM
  role: Used to simulate two million Illumina reads at 15X coverage from the three
    reference genomes for benchmarking
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:b6f6d490569cfc72a234e1a05f92172ba55e5abb00fa2afe93c7ce0441c45177
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.6
n_claims: 10
---

# operon-prediction-pipeline-execution

## Summary

Execute the MetaRon pipeline (github.com/zaidissa/MetaRon) to predict operons from metagenomic shotgun reads or pre-assembled scaftigs without functional or experimental information, using de novo assembly (IDBA), gene prediction (Prodigal/MetaGeneMark), de novo promoter prediction (NNPP 2.2), and co-directional proximon clustering with intergenic distance < 601 bp.

## When to use

Use when input is filtered shotgun FASTQ (or pre-assembled scaftigs plus a Prodigal/MetaGeneMark gene-prediction .tab) from a bacterial genome or metagenome and de novo operon predictions are the target output.

## When NOT to use

- The goal is functional/KEGG annotation or secondary-metabolite analysis rather than operon structure — the article notes secondary metabolite prediction and pathway annotation are not yet integrated into the pipeline and were performed separately
- The environment is not Linux — the README states the pipeline is restricted to Linux and academic use
- A curated reference genome operon annotation already exists and de novo metagenomic prediction is unnecessary

## Inputs

- Filtered paired-end Illumina FASTQ files (metagenomic or whole-genome shotgun reads), supplied as two files (--read_type merge) or one file (--read_type paired)
- Pre-assembled scaftig FASTA file (for --process op mode, via --isc)
- Gene-prediction .tab file from Prodigal or MetaGeneMark (for --process op mode, via --igp)
- Reference operon annotations (e.g., DOOR/ODB/RegulonDB) for downstream benchmarking

## Outputs

- Operon_File.tab — canonical MetaRon operon prediction file listing predicted operons and their member genes
- Intermediate scaftigs (IDBA assembly, filtered to > 500 bp) and gene predictions (Prodigal/MetaGeneMark)
- Benchmark metrics (sensitivity, specificity, accuracy) when compared against reference operon annotations, e.g., simulated_genome_operon_performance.csv or operon_performance_c20.csv

## How to apply

For raw reads, run MetaRon in 'ago' mode (metaron --process ago) supplying --read_type (merge/paired), --read_length (l for >128 bases, r for shorter), and both paired-end FASTQ files; MetaRon assembles scaftigs with IDBA, removes scaftigs <= 500 bp, predicts genes with Prodigal, and runs de novo promoter prediction with NNPP2.2 (its path must be registered in config.txt). For pre-assembled data, use 'op' mode with --igp (gene-prediction .tab), --isc (scaftig FASTA), and --tool 2 for Prodigal or 1 for MetaGeneMark. Operons are called by identifying co-directional proximal gene clusters (proximons) with intergenic distance < 601 bp, merging them into operons written to Operon_File.tab. Validate by matching predicted operons against reference annotations (e.g., DOOR/ODB/RegulonDB) at the gene-pair level and computing sensitivity (TP/(TP+FN)), specificity (TN/(TN+FP)), and accuracy; reported benchmarks are 93.7/75.5/88.1% on a simulated three-genome mixture and 87/91/88% on the E. coli c20 draft genome.

## Related tools

- **MetaRon** (Master pipeline orchestrating assembly, gene prediction, promoter prediction, proximon identification, and operon prediction; installed via pip install metaron) — github.com/zaidissa/MetaRon
- **IDBA** (De novo assembles filtered shotgun reads into scaftigs inside the 'ago' process) — github.com/zaidissa/MetaRon
- **prodigal** (Predicts genes on filtered scaftigs (selected with --tool 2 in 'op' mode); alternative gene caller is MetaGeneMark (--tool 1)) — github.com/zaidissa/MetaRon
- **NNPP** (Performs de novo promoter prediction (BDGP Neural Network Promoter Prediction 2.2); its executable path must be registered in config.txt)
- **NeSSM** (Used to simulate two million Illumina reads at 15X coverage from the three reference genomes for benchmarking)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output ~/path/to/output/directory/
```

## Evaluation signals

- Operon_File.tab exists and is well-formed, reporting operonic information for the sample
- On a simulated three-genome mixture (E. coli MG1655, M. tuberculosis H37Rv, B. subtilis 168), gene-pair-level benchmarking against DOOR should yield sensitivity ~93.7%, specificity ~75.5%, accuracy ~88.1% (within ~±1 percentage point)
- On the E. coli c20 draft genome (GenBank NGBR00000000.1), benchmarking should reproduce sensitivity 87%, specificity 91%, accuracy 88%
- Scaftigs shorter than or equal to 500 bp are absent from the intermediate assembly (filter step)
- Predicted proximons satisfy co-directionality and intergenic distance < 601 bp thresholds

## Limitations

- NNPP's promoter prediction can fail to identify promoters ('the inability of the NNPP tool to identify the promoter'), affecting operon calls
- Secondary metabolite prediction, pathway annotation, and graphical representation are not integrated; analysis of predicted operons was performed separately
- Pipeline restricted to Linux and academic use only
- For 'op' mode, the scaftig FASTA and gene-prediction .tab must match the exact formats documented in the README
- Python 2.7 is required per the README prerequisites (the article index mentions Python > 3.0), so environment compatibility must be checked

## Evidence

- [abstract] MetaRon consistently achieved high operon prediction sensitivity, specificity and accuracy across E. coli whole-genome (97.8, 94.1 and 92.4%), simulated genome (93.7, 75.5 and 88.1%) and E. coli c20: "MetaRon consistently achieved high operon prediction sensitivity, specificity and accuracy across E. coli whole-genome (97.8, 94.1 and 92.4%), simulated genome (93.7, 75.5 and 88.1%) and E. coli c20"
- [results] proximoń IGD threshold and gene count: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [results] scaftig filter step: "Scaftigs with length less than or equal to 500 bp were removed."
- [readme] README ago-mode invocation: "metaron --sample ERR022075 --process ago --read_type merge OR paired --read_length r OR l --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output"
- [readme] README op-mode parameters: "If metagenomic scaffolds and gene predictions are already available, the user can predict operon under the process "op""
- [readme] README NNPP config requirement: "2- Add NNPP2.2 path to the config.txt file"
