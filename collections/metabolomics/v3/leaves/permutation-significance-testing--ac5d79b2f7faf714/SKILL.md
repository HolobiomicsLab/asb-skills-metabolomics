---
name: permutation-significance-testing
description: Use when correlating metabolomics-derived substructure motifs (Mass2Motifs)
  with genome-mined sub-cluster features across a paired genome-metabolome strain
  panel, and you need p-values that account for non-independence of closely related
  strains.
when_to_use_negative:
- The input is a single BGC–molecule pair without a replicated strain panel — permutation
  of co-occurrence across strains requires multiple genomes and metabolomes.
- Correlations are between variables expected to be independent (e.g. two mass motifs);
  the null here is specifically about BGC-required-but-possibly-cryptic co-occurrence.
- Your strain panel contains only highly clonal isolates and you cannot correct for
  shared-BGC inflation — the p-values would be spuriously small.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iPRESTO
  role: Generates the sub-cluster motifs and SCCs that are one side of each tested
    correlation pair, via PRESTO-STAT and PRESTO-TOP
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Identifies the BGCs (5,927 BGCs from 146 Streptomyces/Salinispora strains,
    antiSMASH 3.0) whose per-strain presence feeds the co-occurrence correlations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:d265c58b5f9c2cebbcc836244c019f842377fa4d0e761d5ca0b9035640450aa4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mass2Motifs
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# permutation-significance-testing

## Summary

Assess whether observed co-occurrence correlation scores between MS/MS-derived Mass2Motifs and genome-mined biosynthetic sub-clusters (iPRESTO sub-cluster motifs and sub-cluster clans/SCCs) exceed chance, by permutation testing that explicitly controls for inflation from highly related strains sharing many BGCs and compounds.

## When to use

Use when correlating metabolomics-derived substructure motifs (Mass2Motifs) with genome-mined sub-cluster features across a paired genome-metabolome strain panel, and you need p-values that account for non-independence of closely related strains.

## When NOT to use

- The input is a single BGC–molecule pair without a replicated strain panel — permutation of co-occurrence across strains requires multiple genomes and metabolomes.
- Correlations are between variables expected to be independent (e.g. two mass motifs); the null here is specifically about BGC-required-but-possibly-cryptic co-occurrence.
- Your strain panel contains only highly clonal isolates and you cannot correct for shared-BGC inflation — the p-values would be spuriously small.

## Inputs

- Mass2Motif presence/absence (or intensity-derived) matrix per strain from MS/MS metabolomics of the 145 actinobacteria
- iPRESTO sub-cluster motif occurrence table per strain (from querying 5,927 antiSMASH 3.0-predicted BGCs against antiSMASH-DB sub-cluster motifs)
- iPRESTO sub-cluster clan (SCC) occurrence table per strain
- strain taxonomy/phylogeny information needed to model the inflation from related strains sharing BGCs and compounds

## Outputs

- Table of Doroghazi-style correlation scores for all 107,590 Mass2Motif–sub-cluster motif pairs
- Table of correlation scores for all 122,404 Mass2Motif–SCC pairs
- Permutation p-value column for every pair, with significance flags; reported result: 3,230 significant Mass2Motif–sub-cluster motif pairs and 1,939 significant Mass2Motif–SCC pairs
- Highlighted high-score/low-p links, e.g. the five links between staurosporine-related Mass2Motifs and amino-sugar sub-cluster motifs/SCCs (Fig 6)

## How to apply

First compute co-occurrence correlation scores (Doroghazi et al. style) for each Mass2Motif–sub-cluster motif pair (107,590 pairs) and each Mass2Motif–SCC pair (122,404 pairs) across the 145-actinobacteria panel, under the assumption that a BGC is required for synthesis but may be cryptic. Then run permutation tests on every pair to estimate how likely a high correlation score is by chance, permuting the strain labels so that the null preserves the structure of the data. Critically, the permutation scheme must control for inflation caused by highly related strains sharing many BGCs and compounds — naive per-strain shuffling will overstate significance. Finally assemble a table of correlation scores with permutation p-values for all pairs, flagging the significant ones for downstream metabologenomic linking.

## Related tools

- **iPRESTO** (Generates the sub-cluster motifs and SCCs that are one side of each tested correlation pair, via PRESTO-STAT and PRESTO-TOP) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Identifies the BGCs (5,927 BGCs from 146 Streptomyces/Salinispora strains, antiSMASH 3.0) whose per-strain presence feeds the co-occurrence correlations)

## Evaluation signals

- The results table reports exactly 3,230 'significant' Mass2Motif–sub-cluster motif pairs after permutation testing.
- The results table reports exactly 1,939 'significant' Mass2Motif–SCC pairs.
- Five high correlation scores with low p-values are found between the two staurosporine-related Mass2Motifs and the amino-sugar sub-cluster motifs/SCCs, matching Fig 6.
- Permutation p-values are computed for all 107,590 motif-pair and 122,404 SCC-pair tests, not only for a pre-selected subset.
- The permutation scheme demonstrably accounts for highly related strains sharing many BGCs and compounds (e.g. p-value distribution shows no inflation from clonal structure).

## Limitations

- Correlation assumes a BGC is required for synthesis but may be cryptic; silent/cryptic BGC–motif pairs inflate the testable pair space (~107k–122k pairs), diluting power.
- Significance is inflated by highly related strains sharing many BGCs and compounds; without correcting the permutation scheme, p-values are unreliable.
- Version numbers for antiSMASH and iPRESTO are not stated in the supplementary material, complicating exact reproduction.

## Evidence

- [abstract] Correlating sub-clusters to MS/MS substructure motifs across the 145-actinobacteria panel is the biological context that motivates the permutation test.: "for a collection of 145 actinobacteria how substructures can aid in linking BGCs to molecules by correlating iPRESTO-detected sub-clusters to MS/MS-derived Mass2Motifs substructure patterns"
- [supplementary] The quantities being permutation-tested are the correlation scores between Mass2Motifs and sub-clusters.: "Correlation scores between Mass2Motifs and sub-clusters"
- [intro] iPRESTO supplies the sub-cluster features whose co-occurrence with Mass2Motifs is tested for significance.: "we propose an improved scalable method for unsupervised sub-cluster prediction which we called the integrated Prediction and Rigorous Exploration of biosynthetic Sub-clusters Tool (iPRESTO)"
- [readme] build_subpfam defines the per-clade subPfam features underlying iPRESTO tokenisation; its tree-cutting parameter is a key method input.: [evidence span withheld — anchor status: external_doc]
