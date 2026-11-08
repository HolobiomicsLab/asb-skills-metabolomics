---
name: paired-omics-bgc-metabolite-linking-workflow
description: 'Use when you have paired genomic and metabolomic data from the same
  microbial strains and want to link biosynthetic gene clusters (BGCs) to the metabolite
  features they plausibly encode — mine BGCs from assembled genomes with antiSMASH,
  tokenize BGC domains with iPRESTO/Pfam and cluster them into gene cluster families
  (GCFs) with BiG-SCAPE, build an MS/MS molecular network from paired LC-MS/MS data
  with GNPS to obtain molecular families (MFs), then score GCF-MF co-occurrence across
  strains (NPLinker-style Metcalf/hypergeometric scoring) to produce a ranked table
  of candidate BGC-metabolite links for natural-product discovery.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 6
  member_skills:
  - biosynthetic-gene-cluster-prediction-antismash
  - biosynthetic-gene-cluster-annotation-parsing
  - genome-mining-bgc-detection
  - biosynthetic-domain-curation-via-ec-annotation
  - pfam-domain-tokenisation
  - biosynthetic-gene-domain-tokenisation
  - presto-top-motif-matching
  - biosynthetic-gene-cluster-similarity-network-analysis
  - gcf-prevalence-distribution-analysis-across-genomes
  - gene-cluster-family-grouping
  - gcf-genome-based-class-annotation
  - gnps-molecular-networking-workflow
  - feature-based-molecular-networking--7b025a18b065ae1c
  - gcms-molecular-networking
  - feature-based-molecular-networking--a1d4da5bf30d5f86
  - gcf-molecular-family-link-scoring
  - metcalf-co-occurrence-scoring
  - metcalf-strain-correlation-scoring
  - hypergeometric-score-standardisation
  - validated-link-rank-evaluation
  - bgc-msms-candidate-link-scoring
  - bgc-spectrum-link-ranking
  member_tools:
  - BiG-SCAPE
  - MIBiG
  - antiSMASH v4
  - ECDomainMiner
  - antiSMASH
  - build_subpfam
  - hmmscan
  - iPRESTO
  - HMMER 3.1b2
  - hmmalign
  - GNPS
  - MSConvert (ProteoWizard)
  - MZmine2
  - OpenMS TOPPView
  - BigScape
  - NPLinker
  - Python
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Paired-Omics BGC <-> Metabolite Linking (genome mining + molecular networking -> ranked links)

## Summary

Assembled genomes + paired LC-MS/MS in, a ranked BGC-metabolite link table out: antiSMASH BGC detection, iPRESTO/Pfam domain tokenization, BiG-SCAPE gene cluster family clustering, GNPS molecular networking, and NPLinker-style genomic-metabolomic co-occurrence scoring.


## When to use

Use when you have paired genomic and metabolomic data from the same microbial strains and want to link biosynthetic gene clusters (BGCs) to the metabolite features they plausibly encode — mine BGCs from assembled genomes with antiSMASH, tokenize BGC domains with iPRESTO/Pfam and cluster them into gene cluster families (GCFs) with BiG-SCAPE, build an MS/MS molecular network from paired LC-MS/MS data with GNPS to obtain molecular families (MFs), then score GCF-MF co-occurrence across strains (NPLinker-style Metcalf/hypergeometric scoring) to produce a ranked table of candidate BGC-metabolite links for natural-product discovery.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — bgc_mine

**Goal:** assembled genomes -> detected BGCs (antiSMASH genome mining)

**EDAM operation:** http://edamontology.org/operation_0362

**Inputs:** fasta · **Outputs:** genbank/antismash-bgc

**Candidate leaf skills:** `biosynthetic-gene-cluster-prediction-antismash` (primary), `biosynthetic-gene-cluster-annotation-parsing`, `genome-mining-bgc-detection`

**Tools (primary):** BiG-SCAPE, MIBiG, antiSMASH v4

**Other candidate tools:** HMMER 3.1b2, antiSMASH, hmmalign, build_subpfam, iPRESTO

**Grounding:** 2 KB(s); DOIs: 10.1038/s41589-019-0400-9, 10.1371/journal.pcbi.1010462

### Stage 2 — bgc_tokenize

**Goal:** GenBank BGC records -> Pfam-domain tokens and sub-cluster motifs (iPRESTO)

**EDAM operation:** http://edamontology.org/operation_0306

**Inputs:** genbank/antismash-bgc · **Outputs:** tsv/bgc-domain-tokens

**Candidate leaf skills:** `biosynthetic-domain-curation-via-ec-annotation` (primary), `pfam-domain-tokenisation`, `biosynthetic-gene-domain-tokenisation`, `presto-top-motif-matching`

**Tools (primary):** ECDomainMiner, antiSMASH, build_subpfam, hmmscan, iPRESTO

**Other candidate tools:** Pfam, SubClusterBlast

**Grounding:** 1 KB(s); DOIs: 10.1371/journal.pcbi.1010462

### Stage 3 — gcf_cluster

**Goal:** BGCs -> gene cluster families (BiG-SCAPE similarity clustering)

**EDAM operation:** http://edamontology.org/operation_3432

**Inputs:** genbank/antismash-bgc, tsv/bgc-domain-tokens · **Outputs:** tsv/gcf-table

**Candidate leaf skills:** `biosynthetic-gene-cluster-similarity-network-analysis` (primary), `gcf-prevalence-distribution-analysis-across-genomes`, `gene-cluster-family-grouping`, `gcf-genome-based-class-annotation`

**Tools (primary):** BiG-SCAPE, HMMER 3.1b2, MIBiG, antiSMASH, hmmalign

**Other candidate tools:** antiSMASH v4, BiG-MAP, MASH, bigmap-analyse, bigmap-family, bigmap-map, gutSMASH, MolNetEnhancer, NPLinker

**Grounding:** 3 KB(s); DOIs: 10.1038/s41589-019-0400-9, 10.1128/msystems.00937-21, 10.1186/s40168-022-01444-3

### Stage 4 — mf_network

**Goal:** paired LC-MS/MS spectra -> molecular families (GNPS feature-based molecular networking)

**EDAM operation:** http://edamontology.org/operation_3767

**Inputs:** mgf/gnps-fbmn · **Outputs:** tsv/mf-table

**Candidate leaf skills:** `gnps-molecular-networking-workflow` (primary), `feature-based-molecular-networking--7b025a18b065ae1c`, `gcms-molecular-networking`, `feature-based-molecular-networking--a1d4da5bf30d5f86`

**Tools (primary):** GNPS, MSConvert (ProteoWizard), MZmine2, OpenMS TOPPView

**Other candidate tools:** MS-DIAL, MZmine, XCMS, GNPS_Workflows, MSHub, pyMolNetEnhancer, ChemWalker, GNPS2, MSConvert, MZmine 2.53, SIRIUS 5.8.5

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-021-23953-9, 10.1038/s41587-020-0700-3, 10.1038/s41596-020-0317-5, 10.1101/2024.08.07.607073

### Stage 5 — link_score

**Goal:** GCFs + MFs + strain co-occurrence -> ranked GCF-MF links (NPLinker Metcalf/hypergeometric scoring)

**EDAM operation:** http://edamontology.org/operation_3357

**Inputs:** tsv/gcf-table, tsv/mf-table · **Outputs:** tsv/gcf-mf-scores

**Candidate leaf skills:** `gcf-molecular-family-link-scoring` (primary), `metcalf-co-occurrence-scoring`, `metcalf-strain-correlation-scoring`, `hypergeometric-score-standardisation`

**Tools (primary):** BigScape, GNPS, NPLinker, Python

**Other candidate tools:** BiG-SCAPE, AntiSMASH

**Grounding:** 2 KB(s); DOIs: 10.1186/s40168-022-01444-3, 10.1371/journal.pcbi.1008920

### Stage 6 — report

**Goal:** consolidate scored links + provenance into a ranked BGC-metabolite link table

**EDAM operation:** http://edamontology.org/operation_3695

**Inputs:** tsv/gcf-mf-scores, tsv/gcf-table · **Outputs:** tsv

**Candidate leaf skills:** `validated-link-rank-evaluation` (primary), `bgc-msms-candidate-link-scoring`, `bgc-spectrum-link-ranking`

**Tools (primary):** AntiSMASH, BigScape, GNPS, MIBiG, Python, nplinker

**Other candidate tools:** BiG-SCAPE, CANOPUS, NPClassScore

**Grounding:** 2 KB(s); DOIs: 10.1186/s40168-022-01444-3, 10.1371/journal.pcbi.1008920

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
