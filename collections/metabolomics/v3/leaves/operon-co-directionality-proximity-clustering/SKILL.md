---
name: operon-co-directionality-proximity-clustering
description: Use when you need to predict operons from raw shotgun FASTQ reads or
  from pre-assembled scaftigs with gene predictions, and no functional/experimental
  operon information is available.
when_to_use_negative:
- You already have curated, experimentally supported operon annotations for the organism
  — use those instead of de novo proximon calling
- Input data is functional/annotation tables rather than shotgun reads or scaftigs
  + gene predictions — MetaRon cannot operate without sequence data
- Read lengths exceed 128 bases and you pass the wrong --read_length flag ('l' vs
  'r'), or reads are paired in one file but 'merge' is set — the pipeline parameters
  would be invalid
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaRon
  role: 'End-to-end operon prediction pipeline: proximon identification from co-directionality,
    IGD and promoter evidence; emits Operon_File.tab'
  repo: https://github.com/zaidissa/MetaRon
- name: IDBA
  role: De novo assembly of raw reads into scaftigs before gene and operon prediction
    (82 scaftigs in the benchmark run)
- name: Prodigal
  role: Gene prediction on filtered scaftigs (4227 genes in the benchmark run); selected
    with --tool 2 in 'op' mode
- name: NNPP
  role: BDGP Neural Network Promoter Prediction 2.2, used for de novo promoter prediction
    upstream/downstream of genes to support operon calls; path must be registered
    in config.txt
- name: Python > 3.0
  role: Programming language for MetaRon pipeline execution and downstream metric
    computation
  repo: https://github.com/zaidissa/MetaRon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:9f902783183725f092f8d7793b34170ea8e574d189cfc31bc18ad9dddc52d934
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - Prodigal
  - Python
  - Python > 3.0
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# operon-co-directionality-proximity-clustering

## Summary

Predict bacterial operons from shotgun sequencing data using MetaRon, which clusters co-directional gene pairs (proximons) by intergenic distance (IGD < 601 bp) and promoter presence/absence, without requiring functional or experimental annotations. The skill reproduces the MetaRon benchmark of 97.8% sensitivity, 94.1% specificity and 92.4% accuracy on E. coli K-12 MG1655 HiSeq reads.

## When to use

Use when you need to predict operons from raw shotgun FASTQ reads or from pre-assembled scaftigs with gene predictions, and no functional/experimental operon information is available.

## When NOT to use

- You already have curated, experimentally supported operon annotations for the organism — use those instead of de novo proximon calling
- Input data is functional/annotation tables rather than shotgun reads or scaftigs + gene predictions — MetaRon cannot operate without sequence data
- Read lengths exceed 128 bases and you pass the wrong --read_length flag ('l' vs 'r'), or reads are paired in one file but 'merge' is set — the pipeline parameters would be invalid

## Inputs

- Paired-end Illumina HiSeq FASTQ reads (e.g. E. coli K-12 MG1655, SRA accession SRP029211), either split into two files ('merge') or combined in one file ('paired')
- Pre-assembled scaftig FASTA and Prodigal or MetaGeneMark gene prediction .tab file (for 'op' process mode)
- Reference operon annotations (e.g. DOOR operons for E. coli K-12 MG1655) for benchmarking
- Reference genome (e.g. E. coli MG1655 NC_000913.3) for coordinate mapping

## Outputs

- Operon_File.tab — MetaRon's predicted operon calls with operonic information
- Intermediate scaftigs and Prodigal gene predictions (benchmark run: 82 scaftigs, 4227 genes)
- Sensitivity/specificity/accuracy metrics versus a reference operon database (benchmark: 97.8, 94.1, 92.4%)

## How to apply

Run MetaRon in 'ago' process mode (metaron --sample <name> --process ago --read_type merge --read_length r --paired_1 R1.fastq --paired_2 R2.fastq --output <outdir>), which chains IDBA de novo assembly, removal of scaftigs ≤ 500 bp, Prodigal gene prediction, NNPP 2.2 promoter prediction (register the NNPP path in config.txt) and operon calling. For already-assembled data, use 'op' mode with a gene .tab file (--igp), a scaftig file (--isc) and --tool 2 for Prodigal. The operon logic groups co-directional adjacent genes into proximons using an intergenic distance threshold (IGD < 601 bp in the article; README states ≤ 600 bp) and requires promoter prediction via BDGP NNPP 2.2 to confirm upstream/downstream promoter presence. Set --read_type 'merge' vs 'paired' per FASTQ layout and --read_length 'r' vs 'l' depending on whether reads are shorter than 128 bases. Validate by mapping predicted gene pairs to a reference genome (e.g. E. coli MG1655 NC_000913.3) and comparing coordinate overlap against reference operons such as DOOR, computing sensitivity, specificity and accuracy. The published E. coli K-12 MG1655 benchmark expects 82 scaftigs, 4227 predicted genes, 822 co-directional proximal clusters (2955 genes), and metrics of 97.8/94.1/92.4%.

## Related tools

- **MetaRon** (End-to-end operon prediction pipeline: proximon identification from co-directionality, IGD and promoter evidence; emits Operon_File.tab) — https://github.com/zaidissa/MetaRon
- **IDBA** (De novo assembly of raw reads into scaftigs before gene and operon prediction (82 scaftigs in the benchmark run))
- **Prodigal** (Gene prediction on filtered scaftigs (4227 genes in the benchmark run); selected with --tool 2 in 'op' mode)
- **NNPP** (BDGP Neural Network Promoter Prediction 2.2, used for de novo promoter prediction upstream/downstream of genes to support operon calls; path must be registered in config.txt)
- **Python > 3.0** (Programming language for MetaRon pipeline execution and downstream metric computation) — https://github.com/zaidissa/MetaRon

## Examples

```
metaron --sample SRP029211 --process ago --read_type merge --read_length r --paired_1 R1.fastq --paired_2 R2.fastq --output ~/outdir/
```

## Evaluation signals

- Computed sensitivity/specificity/accuracy against DOOR reference operons match the published E. coli whole-genome benchmark of 97.8%, 94.1% and 92.4%
- Sanity-check intermediate scale of the E. coli K-12 MG1655 HiSeq run: 82 scaftigs (after removing scaftigs ≤ 500 bp), 4227 Prodigal genes, 822 co-directional proximal clusters (IGD < 601 bp) containing 2955 genes
- All genes within each predicted operon are on the same strand (co-directionality invariant)
- Operon_File.tab is produced and parses as the documented output of the 'ago' or 'op' process
- Read_length and read_type flags match the FASTQ layout (merge/paired, reads shorter/longer than 128 bases)

## Limitations

- NNPP may fail to identify promoters, affecting operon calls ('the inability of the NNPP tool to identify the promoter')
- Pipeline is restricted to Linux and academic use only
- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated into the pipeline (planned future work); downstream analysis must be run separately on predicted operons
- Promoter-based operon redefinition uses an IGD threshold of ≤ 600 bp per the README (601 bp in the article results), so cluster counts depend on this cutoff
- No wet-lab validation was performed; findings would require wet-lab resources

## Evidence

- [abstract] MetaRon consistently achieved high operon prediction sensitivity, specificity and accuracy across E. coli whole-genome (97.8, 94.1 and 92.4%): "MetaRon consistently achieved high operon prediction sensitivity, specificity and accuracy across E. coli whole-genome (97.8, 94.1 and 92.4%)"
- [results] MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes.: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [intro] The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene: "The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene"
- [readme] Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp.: "Metagenomic operon prediction redefines the operonic clusters by identifying promoters in co-directional genes with an intergenic distance threshold of <= 600 bp."
- [results] Scaftigs with length less than or equal to 500 bp were removed.: "Scaftigs with length less than or equal to 500 bp were removed."
