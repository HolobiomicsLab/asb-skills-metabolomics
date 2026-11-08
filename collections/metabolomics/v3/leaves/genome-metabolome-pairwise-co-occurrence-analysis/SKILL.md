---
name: genome-metabolome-pairwise-co-occurrence-analysis
description: Use when a paired genome-metabolome dataset exists (BGCs called per strain
  plus MS/MS-derived Mass2Motifs per strain) and the goal is to statistically link
  sub-clusters or sub-cluster clans (SCCs) to molecular substructures.
when_to_use_negative:
- The genome and metabolome data are not paired by strain/organism — co-occurrence
  across strains cannot be computed without a shared mapping of BGCs and Mass2Motifs
  to the same strains.
- You need de novo sub-cluster prediction itself rather than linking of already-predicted
  sub-clusters to metabolites — that is the iPRESTO sub-cluster mining step (PRESTO-STAT/PRESTO-TOP),
  not this correlation skill.
- Mass2Motif assignments or BGC calls are unavailable per strain (e.g. only unlinked
  genome assemblies and MS feature tables with no strain correspondence).
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: iPRESTO
  role: Queries predicted BGCs against antiSMASH-DB sub-cluster motifs and sub-cluster
    clans (SCCs) to produce the sub-cluster presence/absence data correlated with
    Mass2Motifs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Identifies the per-strain BGCs (e.g. 5,927 BGCs in 146 Streptomyces/Salinispora
    strains with antiSMASH 3.0) that form the genome side of the correlation
- name: Mass2Motifs
  role: MS/MS-derived substructure patterns providing the metabolome side of the pairwise
    co-occurrence analysis
- name: build_subpfam
  role: Builds per-clade subPfam pHMMs from aligned protein sequences, which underpins
    the tokenised BGC sub-cluster features used in the correlation
  repo: https://github.com/satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:dc6608939ac1e4e42d9f0b7457099c0970b0da82a3aa2ba8d13fc6932dce6506
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Genome-metabolome pairwise co-occurrence analysis

## Summary

Links mass-spectrometry-derived molecular substructures (Mass2Motifs) to genome-mined biosynthetic sub-clusters by computing co-occurrence correlation scores across a paired strain set, then assessing significance with permutation testing that controls for phylogenetic inflation from closely related strains.

## When to use

Use when a paired genome-metabolome dataset exists (BGCs called per strain plus MS/MS-derived Mass2Motifs per strain) and the goal is to statistically link sub-clusters or sub-cluster clans (SCCs) to molecular substructures.

## When NOT to use

- The genome and metabolome data are not paired by strain/organism — co-occurrence across strains cannot be computed without a shared mapping of BGCs and Mass2Motifs to the same strains.
- You need de novo sub-cluster prediction itself rather than linking of already-predicted sub-clusters to metabolites — that is the iPRESTO sub-cluster mining step (PRESTO-STAT/PRESTO-TOP), not this correlation skill.
- Mass2Motif assignments or BGC calls are unavailable per strain (e.g. only unlinked genome assemblies and MS feature tables with no strain correspondence).

## Inputs

- Per-strain BGC annotations (e.g. 5,927 antiSMASH 3.0-predicted BGCs from 146 Streptomyces/Salinispora strains)
- iPRESTO sub-cluster motif and SCC presence/absence per strain, from querying BGCs against antiSMASH-DB sub-cluster motifs and SCCs
- Mass2Motif occurrence per strain from MS/MS molecular networking (metabolomics-derived substructure patterns)
- Strain phylogenetic/relationship information used to inform permutation testing against inflation from closely related strains

## Outputs

- Table of co-occurrence correlation scores and permutation p-values for all Mass2Motif–sub-cluster motif pairs (107,590 pairs)
- Table of correlation scores and permutation p-values for all Mass2Motif–SCC pairs (122,404 pairs)
- List of statistically significant pairs (3,230 significant Mass2Motif–sub-cluster motif pairs; 1,939 significant Mass2Motif–SCC pairs) with biological validation highlights (e.g. staurosporine Mass2Motifs vs amino-sugar sub-clusters)

## How to apply

First obtain per-strain BGC predictions (e.g. the 5,927 BGCs from 146 Streptomyces/Salinispora strains identified with antiSMASH 3.0) and query all BGCs with iPRESTO against antiSMASH-DB sub-cluster motifs and sub-cluster clans (SCCs). For every Mass2Motif–sub-cluster pair (107,590 pairs in the reference study) and every Mass2Motif–SCC pair (122,404 pairs), compute the Doroghazi et al. correlation score that measures how frequently the mass motif and sub-cluster co-occur across strains, under the assumption that a BGC is required for synthesis of the substructure but may be cryptic. Because highly related strains share many BGCs and compounds and would inflate apparent co-occurrence, run permutation tests on all pairs to estimate the chance probability of high correlation scores. Assemble a table of correlation scores and permutation p-values, and report pairs passing significance (the reference analysis found 3,230 significant Mass2Motif–sub-cluster motif pairs and 1,939 significant Mass2Motif–SCC pairs). Validate biologically: e.g. the two staurosporine-related Mass2Motifs should show high correlation with low p-values against amino-sugar sub-cluster motifs/SCCs (Fig 6).

## Related tools

- **iPRESTO** (Queries predicted BGCs against antiSMASH-DB sub-cluster motifs and sub-cluster clans (SCCs) to produce the sub-cluster presence/absence data correlated with Mass2Motifs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Identifies the per-strain BGCs (e.g. 5,927 BGCs in 146 Streptomyces/Salinispora strains with antiSMASH 3.0) that form the genome side of the correlation)
- **Mass2Motifs** (MS/MS-derived substructure patterns providing the metabolome side of the pairwise co-occurrence analysis)
- **build_subpfam** (Builds per-clade subPfam pHMMs from aligned protein sequences, which underpins the tokenised BGC sub-cluster features used in the correlation) — https://github.com/satriaphd/build_subpfam

## Evaluation signals

- The permutation-corrected analysis reports 3,230 significant Mass2Motif–sub-cluster motif pairs and 1,939 significant Mass2Motif–SCC pairs.
- Correlation scores are computed for the full pair space (107,590 Mass2Motif–sub-cluster motif pairs; 122,404 Mass2Motif–SCC pairs), not only pre-selected candidates.
- Permutation testing is present and controls for inflation from highly related strains sharing many BGCs and compounds.
- Known positive controls validate: the two staurosporine-related Mass2Motifs show high correlation with low p-values against amino-sugar sub-cluster motifs/SCCs (Fig 6).
- Sub-clusters of length one are excluded from the correlated features as they are treated as noise.

## Limitations

- Correlation is inflated by highly related strains sharing many BGCs and compounds, so permutation testing that accounts for this relatedness is mandatory; naive co-occurrence counts overstate significance.
- The correlation model assumes a BGC is required for substructure synthesis but that it may be cryptic — silent BGCs may genuinely synthesise the moiety without detectable production, complicating interpretation.
- Sub-clusters of length one are almost certainly noise and must not be counted as correlated features.
- Version provenance matters: the reference dataset used antiSMASH 3.0 BGCs, and no version numbers are stated for iPRESTO in the supplementary material.

## Evidence

- [abstract] Sub-clusters were correlated to MS/MS-derived Mass2Motifs for 145 actinobacteria: "for a collection of 145 actinobacteria how substructures can aid in linking BGCs to molecules by correlating iPRESTO-detected sub-clusters to MS/MS-derived Mass2Motifs substructure patterns"
- [intro] Chemical moieties identified from fragments in mass spectrometry (MS) data could be linked to sub-clusters responsible for their synthesis, as part of MS-guided genome mining strategies [10,12,13].: "Chemical moieties identified from fragments in mass spectrometry (MS) data could be linked to sub-clusters responsible for their synthesis, as part of MS-guided genome mining strategies"
- [supplementary] Correlation scores between Mass2Motifs and sub-clusters: "Correlation scores between Mass2Motifs and sub-clusters"
- [supplementary] not counting sub-clusters of length one as noise: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
