---
name: mass2motif-substructure-linking
description: Use when a strain collection has both genome-mined BGCs (antiSMASH) and
  MS/MS-derived Mass2Motifs per strain, and you need to test which sub-clusters chemically
  explain which molecular substructures.
when_to_use_negative:
- When you only have MS/MS data (Mass2Motifs) without corresponding genome sequences
  / BGC annotations — no sub-clusters exist to correlate against.
- When the goal is de novo BGC detection rather than linking already-mined sub-clusters
  to molecular substructures; use iPRESTO sub-cluster prediction itself instead.
- When strains are few or highly redundant such that permutation testing cannot meaningfully
  control for related-strain co-occurrence inflation.
edam_topics:
- http://edamontology.org/topic_3855
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: iPRESTO
  role: Predicts sub-cluster motifs and SCCs from tokenised BGCs (PRESTO-STAT and
    PRESTO-TOP) that are the genome-side entities correlated with Mass2Motifs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Mines the BGCs per strain whose (sub)Pfam tokenisation feeds iPRESTO sub-cluster
    prediction
- name: Mass2Motifs
  role: MS/MS-derived substructure patterns per strain that form the metabolomics-side
    entities in the correlation analysis
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models from aligned protein sequences used to
    tokenise BGC genes
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:44f577908f34029a63609cb652d022c2f12b4668459bbf9df30ac55cc9b2201b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mass2Motifs
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3855
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mass2motif-substructure-linking

## Summary

Correlates MS/MS-derived Mass2Motif substructure patterns with iPRESTO-predicted biosynthetic sub-clusters (sub-cluster motifs and SCCs) across a paired genome-metabolome strain collection, using a Dorogui-style co-occurrence correlation score plus permutation testing to identify significant metabologenomic links.

## When to use

Use when a strain collection has both genome-mined BGCs (antiSMASH) and MS/MS-derived Mass2Motifs per strain, and you need to test which sub-clusters chemically explain which molecular substructures.

## When NOT to use

- When you only have MS/MS data (Mass2Motifs) without corresponding genome sequences / BGC annotations — no sub-clusters exist to correlate against.
- When the goal is de novo BGC detection rather than linking already-mined sub-clusters to molecular substructures; use iPRESTO sub-cluster prediction itself instead.
- When strains are few or highly redundant such that permutation testing cannot meaningfully control for related-strain co-occurrence inflation.

## Inputs

- antiSMASH-predicted BGCs per strain (e.g. 5,927 BGCs from 146 Streptomyces/Salinispora strains, antiSMASH 3.0 output)
- Tokenised BGC gene sequences as (sub)Pfam domains (non-biosynthetic Pfams removed)
- iPRESTO sub-cluster motifs and sub-cluster clans (SCCs) queried against antiSMASH-DB motifs
- Mass2Motif presence/absence per strain derived from MS/MS molecular networking

## Outputs

- Table of Dorogui correlation scores and permutation p-values for all Mass2Motif–sub-cluster motif pairs (e.g. 107,590 pairs)
- Table of correlation scores and permutation p-values for all Mass2Motif–SCC pairs (e.g. 122,404 pairs)
- List of significant Mass2Motif–sub-cluster links (expected: 3,230 significant motif–sub-cluster pairs and 1,939 significant motif–SCC pairs)

## How to apply

Tokenise antiSMASH-predicted BGCs by detecting (sub)Pfams per gene and removing non-biosynthetic Pfams, then run iPRESTO (PRESTO-STAT co-occurrence statistics and PRESTO-TOP LDA topic modelling) to obtain sub-cluster motifs and sub-cluster clans (SCCs) for every strain. Build presence/absence tables of Mass2Motifs and sub-clusters across the strains (e.g. 107,590 Mass2Motif–sub-cluster motif pairs and 122,404 Mass2Motif–SCC pairs in the 145-actinobacteria dataset with 5,927 antiSMASH 3.0 BGCs). For each pair, compute the Dorogui et al. correlation score measuring how frequently the mass motif and sub-cluster co-occur, under the assumption that a BGC is required for compound synthesis but may remain cryptic (absence of the compound does not imply absence of the BGC). Run permutation tests on all pairs, explicitly controlling for score inflation caused by highly related strains sharing many BGCs and compounds, to obtain p-values for every pair. Assemble a table of correlation scores and permutation p-values, retaining the 'significant' pairs for biological interpretation (e.g. staurosporine-related Mass2Motifs against amino-sugar sub-cluster motifs/SCCs).

## Related tools

- **iPRESTO** (Predicts sub-cluster motifs and SCCs from tokenised BGCs (PRESTO-STAT and PRESTO-TOP) that are the genome-side entities correlated with Mass2Motifs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Mines the BGCs per strain whose (sub)Pfam tokenisation feeds iPRESTO sub-cluster prediction)
- **Mass2Motifs** (MS/MS-derived substructure patterns per strain that form the metabolomics-side entities in the correlation analysis)
- **build_subpfam** (Builds per-clade subPfam pHMM models from aligned protein sequences used to tokenise BGC genes) — satriaphd/build_subpfam

## Evaluation signals

- The results table reports 3,230 significant Mass2Motif–sub-cluster motif pairs and 1,939 significant Mass2Motif–SCC pairs after permutation testing.
- Validation anchor: five high correlation scores with low p-values occur between the two staurosporine-related Mass2Motifs and the amino-sugar sub-cluster motifs/SCCs (as in Fig 6).
- iPRESTO upstream sanity check: the method confirmed 83% of experimentally characterised sub-clusters in MIBiG reference BGCs.
- Sub-clusters of length one must be excluded from correlations, as these are treated as almost definitely noise.
- Mass2Motif–sub-cluster pair enumeration must match the expected pair counts (e.g. 107,590 motif–sub-cluster pairs) before permutation testing begins.

## Limitations

- Permutation tests must control for score inflation caused by highly related strains sharing many BGCs and compounds; ignoring this overstates significance.
- Sub-cluster predictions of length one are almost definitely noise and must not be counted.
- Dataset-specific provenance: BGCs were predicted with antiSMASH 3.0 in the 145/146-actinobacteria case study; version numbers for antiSMASH and iPRESTO are not stated in the supplementary section.
- build_subpfam usage documentation is marked 'under construction', so subPfam construction may need additional guidance.

## Evidence

- [abstract] Correlating iPRESTO sub-clusters with Mass2Motifs across 145 actinobacteria is the core of this skill.: "for a collection of 145 actinobacteria how substructures can aid in linking BGCs to molecules by correlating iPRESTO-detected sub-clusters to MS/MS-derived Mass2Motifs substructure patterns"
- [intro] The metabologenomic rationale for linking MS fragments to synthesising sub-clusters.: "Chemical moieties identified from fragments in mass spectrometry (MS) data could be linked to sub-clusters responsible for their synthesis, as part of MS-guided genome mining strategies"
- [supplementary] BGC tokenisation into (sub)Pfams is the preprocessing step before sub-cluster prediction and correlation.: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] Length-one sub-clusters must be excluded as noise when counting co-occurrences.: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [supplementary] The correlation deliverable is a score table between Mass2Motifs and sub-clusters.: "Correlation scores between Mass2Motifs and sub-clusters"
- [readme] The subPfam models used for tokenisation are built from aligned protein sequences.: [evidence span withheld — anchor status: external_doc]
