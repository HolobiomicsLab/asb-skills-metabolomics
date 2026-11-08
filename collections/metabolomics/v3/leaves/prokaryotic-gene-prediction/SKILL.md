---
name: prokaryotic-gene-prediction
description: Use when starting from assembled prokaryotic or metagenomic scaftigs
  (e.g. IDBA output) and gene coordinates/labels are needed downstream for operon,
  proximon, or promoter-based prediction.
when_to_use_negative:
- The input is raw reads rather than assembled scaftigs — use full assembly (IDBA)
  and the MetaRon 'ago' pipeline first
- The genome/transcriptome is eukaryotic — Prodigal/MetaGeneMark prokaryotic gene
  models are inappropriate
- Operon-level output alone is needed and scaftigs plus a compatible gene prediction
  file are unavailable for 'op' mode
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Prodigal
  role: Gene prediction on filtered scaftigs within the MetaRon 'ago' process; selected
    via --tool 2 in 'op' mode
  repo: https://github.com/zaidissa/MetaRon
- name: MetaGeneMark
  role: Alternative gene predictor accepted by MetaRon 'op' mode via --tool 1
- name: IDBA
  role: Produces the scaftigs on which gene prediction is run (82 scaftigs for E.
    coli K-12 MG1655)
  repo: https://github.com/zaidissa/MetaRon
- name: MetaRon
  role: Pipeline wrapping assembly, scaftig filtering (<= 500 bp removal) and Prodigal
    gene prediction before operon calling
  repo: https://github.com/zaidissa/MetaRon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:f0f587f09b02e9c46d5b13b2a43235208a3d5ae40cff3efc1cb90b2175649865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaGeneMark
  - MetaRon
  - NNPP
  - Prodigal
  - Python
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# prokaryotic-gene-prediction

## Summary

Predicts protein-coding genes from de novo assembled prokaryotic/metagenomic scaftigs using Prodigal (or MetaGeneMark), as performed within the MetaRon pipeline prior to operon calling. On the E. coli K-12 MG1655 HiSeq benchmark, 4227 genes were predicted from the retained scaftigs.

## When to use

Use when starting from assembled prokaryotic or metagenomic scaftigs (e.g. IDBA output) and gene coordinates/labels are needed downstream for operon, proximon, or promoter-based prediction.

## When NOT to use

- The input is raw reads rather than assembled scaftigs — use full assembly (IDBA) and the MetaRon 'ago' pipeline first
- The genome/transcriptome is eukaryotic — Prodigal/MetaGeneMark prokaryotic gene models are inappropriate
- Operon-level output alone is needed and scaftigs plus a compatible gene prediction file are unavailable for 'op' mode

## Inputs

- Scaftigs (FASTA) from de novo assembly with IDBA, after filtering scaftigs <= 500 bp
- Pre-existing gene prediction .tab file (Prodigal or MetaGeneMark format) when running MetaRon 'op' mode

## Outputs

- Gene prediction .tab file with gene coordinates on scaftigs (4227 genes for the E. coli K-12 MG1655 benchmark)
- Gene sets used for downstream co-directional proximon clustering (IGD < 601 bp)

## How to apply

After de novo assembly of shotgun reads (e.g. IDBA within MetaRon) and removal of scaftigs with length less than or equal to 500 bp, run a prokaryotic gene caller on the retained scaftigs. In the MetaRon 'ago' process mode, Prodigal is invoked automatically on the scaftigs; in the 'op' (operon-only) mode, supply an existing gene prediction .tab file from MetaGeneMark or Prodigal and indicate which tool produced it via --tool 1 (MetaGeneMark) or --tool 2 (Prodigal). The gene predictions (coordinates plus strand) feed the co-directional proximon identification step, where genes with IGD < 601 bp and matching direction are clustered before NNPP-based promoter checking and operon calling. For the E. coli K-12 MG1655 SRP029211 benchmark, the retained scaftigs yielded 4227 predicted genes, which supported identification of 822 co-directional proximal gene clusters containing 2955 genes.

## Related tools

- **Prodigal** (Gene prediction on filtered scaftigs within the MetaRon 'ago' process; selected via --tool 2 in 'op' mode) — https://github.com/zaidissa/MetaRon
- **MetaGeneMark** (Alternative gene predictor accepted by MetaRon 'op' mode via --tool 1)
- **IDBA** (Produces the scaftigs on which gene prediction is run (82 scaftigs for E. coli K-12 MG1655)) — https://github.com/zaidissa/MetaRon
- **MetaRon** (Pipeline wrapping assembly, scaftig filtering (<= 500 bp removal) and Prodigal gene prediction before operon calling) — https://github.com/zaidissa/MetaRon

## Examples

```
metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 2 --output ~/path/to/output/directory/
```

## Evaluation signals

- Benchmark-scale check: E. coli K-12 MG1655 (SRP029211) run should produce ~82 scaftigs after IDBA and 4227 predicted genes
- Downstream sanity check: gene predictions should yield 822 co-directional proximal gene clusters (IGD < 601 bp) containing 2955 genes
- Operons derived from these gene calls reproduce 97.8% sensitivity, 94.1% specificity and 92.4% accuracy against DOOR for E. coli K-12 MG1655
- Gene prediction .tab file must follow the MetaRon-specified format (Prodigal or MetaGeneMark) for 'op' mode compatibility
- Scaftigs with length <= 500 bp must be excluded before gene prediction

## Limitations

- MetaRon (and hence its gene prediction step) is restricted to Linux and academic use only
- NNPP promoter prediction (a dependency of downstream operon calling) has an 'inability to identify the promoter' in some cases
- If pre-assembled scaftigs are supplied in 'op' mode, they must conform to the scaftig and gene prediction file formats documented in the README

## Evidence

- [results] Prodigal gene prediction feeds proximon clustering: "The remaining scaftigs resulted in 4227 genes, predicted using prodigal [32]."
- [results] Scaftig length filter precedes gene prediction: "Scaftigs with length less than or equal to 500 bp were removed."
- [results] Genes are clustered by co-directionality and intergenic distance: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [readme] op mode accepts a gene prediction .tab from Prodigal or MetaGeneMark: "Select the gene prediction .tab file generated via MetageneMark or Prodigal"
- [intro] Downstream processing includes gene prediction before operon identification: "performs necessary downstream data processing (de novo assembly, gene prediction, de novo promoter prediction and proximon prediction), before identifying the operons"
