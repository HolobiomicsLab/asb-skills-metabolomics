---
name: promoter-prediction-integration
description: Use when predicting operons from metagenomic or whole-genome shotgun
  data where scaftigs and gene predictions (.tab from Prodigal or MetaGeneMark) exist
  and promoter evidence is needed to delimit operon boundaries.
when_to_use_negative:
- Input is filtered raw reads rather than pre-assembled scaftigs plus gene predictions
  — use the 'ago' process (assembly, gene prediction, operon prediction) instead of
  'op'
- Functional or experimental operon information is already available for the genes
  — MetaRon is designed for prediction without functional/experimental data
- The operating environment is not Linux — the pipeline is restricted to Linux and
  academic use
edam_operation: http://edamontology.org/operation_0440
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaRon
  role: Pipeline orchestrating promoter_prediction() and Prom_IGD_Clustering() to
    integrate promoter predictions with proximons and write Operon_File.tab
  repo: https://github.com/zaidissa/MetaRon
- name: NNPP
  role: Neural Network Promoter Prediction (2.2/2.0) run de novo on up to 700 bp upstream
    sequences of proximon genes
- name: prodigal
  role: Produces the gene prediction .tab file (--tool 2) supplied as operon-prediction
    input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:669d2eefe806bfcf91da66c8290c2a55caa50ff40a3713fd48ed3119cce94b47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - MetaRon
  - NNPP
  - prodigal
  edam_operation: http://edamontology.org/operation_0440
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# promoter-prediction-integration

## Summary

Defines transcription unit boundaries within co-directional proximon clusters by running de novo promoter prediction (NNPP 2.2) on up to 700 bp upstream sequences and integrating promoter presence/absence with intergenic distance (IGD < 601 bp) via Prom_IGD_Clustering in MetaRon.

## When to use

Use when predicting operons from metagenomic or whole-genome shotgun data where scaftigs and gene predictions (.tab from Prodigal or MetaGeneMark) exist and promoter evidence is needed to delimit operon boundaries.

## When NOT to use

- Input is filtered raw reads rather than pre-assembled scaftigs plus gene predictions — use the 'ago' process (assembly, gene prediction, operon prediction) instead of 'op'
- Functional or experimental operon information is already available for the genes — MetaRon is designed for prediction without functional/experimental data
- The operating environment is not Linux — the pipeline is restricted to Linux and academic use

## Inputs

- Gene prediction .tab file generated with MetaGeneMark (--tool 1) or Prodigal (--tool 2)
- Scaftigs FASTA file containing assembled metagenomic scaffolds
- NNPP2.2 executable path registered in MetaRon's config.txt

## Outputs

- Operon_File.tab — tab-delimited operon predictions conforming to the scaftig and gene prediction file format documented for the 'op' process

## How to apply

Configure the NNPP2.2 executable path in MetaRon's config.txt, then load the gene prediction .tab file (--igp) and scaftigs FASTA (--isc) under process 'op' (with --tool 1 for MetaGeneMark or 2 for Prodigal). MetaRon's promoter_prediction() extracts up to 700 bp upstream sequences of genes in co-directional proximal clusters and runs NNPP 2.0/2.2 to produce a promoter prediction matrix. Prom_IGD_Clustering() integrates this matrix with the proximon table, using promoter presence/absence upstream and downstream of each gene together with the IGD < 601 bp criterion to define transcription unit boundaries delimited by upstream and downstream promoters. Operons are written to the tab-delimited Operon_File.tab in the output directory. Rationale: co-directionality and intergenic distance alone over-predict operons; promoter signals upstream/downstream of gene pairs refine which proximon members share a transcription unit.

## Related tools

- **MetaRon** (Pipeline orchestrating promoter_prediction() and Prom_IGD_Clustering() to integrate promoter predictions with proximons and write Operon_File.tab) — https://github.com/zaidissa/MetaRon
- **NNPP** (Neural Network Promoter Prediction (2.2/2.0) run de novo on up to 700 bp upstream sequences of proximon genes)
- **prodigal** (Produces the gene prediction .tab file (--tool 2) supplied as operon-prediction input)

## Examples

```
metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 2 --output ~/path/to/output/directory/
```

## Evaluation signals

- Operon_File.tab exists in the specified output directory and is tab-delimited
- Operon records conform to the scaftig and gene prediction file format documented for the 'op' process
- Predicted operon boundaries are consistent with the IGD < 601 bp criterion and promoter presence/absence upstream/downstream of genes
- Reported metrics on comparable data are high (e.g. 97.8/94.1/92.4% sensitivity/specificity/accuracy on E. coli whole-genome; 87/91/88% on E. coli c20)
- Promoter predictions are only expected within co-directional proximal gene clusters; absence of NNPP calls outside these clusters indicates correct scoping

## Limitations

- The NNPP tool can fail to identify promoters, which limits boundary resolution for affected genes
- Pipeline is restricted to Linux and academic use only
- Requires the NNPP2.2 path to be manually added to config.txt before running
- Python version requirements conflict between article (Python > 3.0) and README (Python 2.7)

## Evidence

- [results] Promoter integration defines operon boundaries: "The pipeline performs operon prediction with high sensitivity based on co-directionality, intergenic distance, and presence/absence of a promoter upstream and downstream of a gene"
- [results] IGD threshold defines proximons: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [results] NNPP limitation: "the inability of the NNPP tool to identify the promoter."
- [readme] README: setup requirement: "Add NNPP2.2 path to the config.txt file"
- [readme] README: operon prediction command: "metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 1 OR 2 --output ~/path/to/output/directory/"
