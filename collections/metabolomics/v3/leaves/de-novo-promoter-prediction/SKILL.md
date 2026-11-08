---
name: de-novo-promoter-prediction
description: Use when predicting operons from shotgun metagenome or whole-genome data
  where promoters must be inferred de novo from scaftig/gene-prediction files because
  no functional annotation or experimental promoter data exist.
when_to_use_negative:
- Promoter annotations are already available from curated databases or experiments
  and only operon calling is needed
- Input is raw reads without prior assembly and you only want gene counts — use the
  gene-prediction tools directly rather than promoter/operon prediction
- Secondary metabolite or pathway annotation is the goal — the article notes these
  analyses are performed separately and are not yet integrated into the pipeline
edam_operation: http://edamontology.org/operation_0435
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: NNPP
  role: BDGP Neural Network Promoter Prediction 2.2 — performs the de novo promoter
    predictions on scaftigs; its path must be added to MetaRon's config.txt
- name: MetaRon
  role: Pipeline that orchestrates NNPP2.2 promoter prediction within the proximon/operon
    prediction stages ('ago' and 'op' modes)
  repo: github.com/zaidissa/MetaRon
- name: IDBA
  role: Upstream de novo assembler producing the scaftigs over which promoters are
    predicted (82 scaftigs in the E. coli K-12 MG1655 run)
- name: prodigal
  role: Gene caller (--tool 2) supplying gene coordinates used to define co-directional
    proximal gene pairs evaluated with promoter evidence
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:8204e29fcdff2ddf57d7569f6c3d7b6204957777ce6c6908ed9124150c22cf92
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - NeSSM
  - Prodigal
  - Python
  - prodigal
  edam_operation: http://edamontology.org/operation_0435
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# de-novo-promoter-prediction

## Summary

De novo prediction of promoters from assembled metagenomic or whole-genome scaftigs using BDGP Neural Network Promoter Prediction 2.2 (NNPP), as embedded in the MetaRon pipeline. It supplies promoter presence/absence evidence used together with gene co-directionality and intergenic distance to call operons from shotgun data without functional or experimental information.

## When to use

Use when predicting operons from shotgun metagenome or whole-genome data where promoters must be inferred de novo from scaftig/gene-prediction files because no functional annotation or experimental promoter data exist.

## When NOT to use

- Promoter annotations are already available from curated databases or experiments and only operon calling is needed
- Input is raw reads without prior assembly and you only want gene counts — use the gene-prediction tools directly rather than promoter/operon prediction
- Secondary metabolite or pathway annotation is the goal — the article notes these analyses are performed separately and are not yet integrated into the pipeline

## Inputs

- Scaftig FASTA file from IDBA assembly (scaftigs > 500 bp), e.g. ERR022075_scaf.fa
- Gene prediction .tab file from Prodigal (--tool 2) or MetaGeneMark (--tool 1), e.g. ERR022075
- Paired-end FASTQ raw reads if running the full 'ago' process from reads
- config.txt with the NNPP2.2 executable path registered

## Outputs

- De novo promoter predictions over scaftigs feeding the proximon stage
- Co-directional proximon clusters (IGD < 601 bp) with promoter evidence
- Operon_File.tab — tab-delimited metagenomic operon predictions

## How to apply

After IDBA assembles filtered raw reads into scaftigs (discarding scaftigs ≤ 500 bp) and Prodigal (or MetaGeneMark) predicts genes, run NNPP2.2 over the scaftigs to detect promoters de novo. In MetaRon, register the NNPP2.2 executable path in the pipeline's config.txt before running, so promoter predictions feed the proximon/operon stage; the promoter signal is combined with co-directionality of adjacent genes and an intergenic distance (IGD) threshold near 600 bp (< 601 bp in the E. coli K-12 MG1655 run) to merge co-directional proximons into operons. For pre-assembled data, use 'metaron --process op' with the gene-prediction .tab (--igp), the scaftig FASTA (--isc) and --tool 1 (MetaGeneMark) or 2 (Prodigal); the promoter step runs automatically once NNPP is configured. Sanity-check against the published E. coli K-12 MG1655 run: 82 scaftigs, 4227 genes, 822 co-directional proximons (IGD < 601 bp) containing 2955 genes.

## Related tools

- **NNPP** (BDGP Neural Network Promoter Prediction 2.2 — performs the de novo promoter predictions on scaftigs; its path must be added to MetaRon's config.txt)
- **MetaRon** (Pipeline that orchestrates NNPP2.2 promoter prediction within the proximon/operon prediction stages ('ago' and 'op' modes)) — github.com/zaidissa/MetaRon
- **IDBA** (Upstream de novo assembler producing the scaftigs over which promoters are predicted (82 scaftigs in the E. coli K-12 MG1655 run))
- **prodigal** (Gene caller (--tool 2) supplying gene coordinates used to define co-directional proximal gene pairs evaluated with promoter evidence)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output ~/path/to/output/directory/
```

## Evaluation signals

- NNPP2.2 path is correctly registered in MetaRon's config.txt (pipeline fails or skips promoter prediction otherwise)
- E. coli K-12 MG1655 HiSeq run reproduces published intermediates: 82 scaftigs, 4227 genes, 822 co-directional proximons (IGD < 601 bp) with 2955 genes
- Downstream operon metrics match reported benchmarks (e.g. 97.8% sensitivity / 94.1% specificity / 92.4% accuracy on E. coli K-12 MG1655 vs DOOR; 93.7/75.5/88.1% on the simulated three-genome mixture)
- Output contains the canonical Operon_File.tab in the specified output directory
- Promoter evidence resolves promoters upstream/downstream of co-directional genes within the ≤ 600 bp IGD window before operons are merged

## Limitations

- The NNPP tool can fail to identify promoters, as noted in the article's results
- Pipeline is restricted to Linux and academic use only
- Only operates downstream of IDBA-assembly and scaftig filtering (≤ 500 bp removal); it does not work on raw reads alone
- Promoter-prediction quality on draft genomes depends on contig quality (E. coli C20 draft genome achieved 87/91/88% sensitivity/specificity/accuracy, lower than the finished E. coli run)

## Evidence

- [intro] performs necessary downstream data processing (de novo assembly, gene prediction, de novo promoter prediction and proximon prediction), before identifying the operons: "performs necessary downstream data processing (de novo assembly, gene prediction, de novo promoter prediction and proximon prediction), before identifying the operons"
- [results] the inability of the NNPP tool to identify the promoter: "the inability of the NNPP tool to identify the promoter"
- [results] MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes.: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [readme] The pipeline predicts metagenomic operons without any any functional or experimental data.: "The pipeline predicts metagenomic operons without any any functional or experimental data."
- [readme] Add NNPP2.2 path to the config.txt file: "2- Add NNPP2.2 path to the config.txt file"
