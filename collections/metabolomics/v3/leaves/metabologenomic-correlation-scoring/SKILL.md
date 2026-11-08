---
name: metabologenomic-correlation-scoring
description: Use when a strain collection has both genome-mined BGCs (with iPRESTO
  sub-cluster motifs or SCCs) and MS/MS data with Mass2Motifs, and the goal is to
  link molecular substructures to their biosynthetic sub-clusters via co-occurrence
  correlation.
when_to_use_negative:
- The genome and metabolome samples are not strain-paired (co-occurrence correlation
  needs both measurements for the same strains).
- The MS/MS data has no Mass2Motif annotations, or the genome mining produced no sub-cluster
  predictions — there is nothing to correlate.
- You only need BGC sub-cluster prediction itself (run iPRESTO alone); this skill
  is specifically the downstream metabologenomic linkage step.
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
tools:
- name: iPRESTO
  role: Query each strain's BGCs against antiSMASH-DB sub-cluster motifs and sub-cluster
    clans (SCCs) to obtain the genome-side features for correlation
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Provides the BGC predictions per strain that are tokenised and queried with
    iPRESTO (antiSMASH 3.0 for the 146-strain dataset)
- name: Mass2Motifs
  role: MS/MS-derived substructure patterns per strain that form the metabolomics-side
    features for correlation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:8c9df141e490ac76a5057162a9a1974a785286f9639694d078ffdf1f613b634b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mass2Motifs
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabologenomic-correlation-scoring

## Summary

Links MS/MS-derived Mass2Motif substructures to genome-mined BGC sub-clusters by computing Doroghazi et al. co-occurrence correlation scores across a strain collection, then applying permutation tests to separate true metabologenomic links from chance co-occurrence.

## When to use

Use when a strain collection has both genome-mined BGCs (with iPRESTO sub-cluster motifs or SCCs) and MS/MS data with Mass2Motifs, and the goal is to link molecular substructures to their biosynthetic sub-clusters via co-occurrence correlation.

## When NOT to use

- The genome and metabolome samples are not strain-paired (co-occurrence correlation needs both measurements for the same strains).
- The MS/MS data has no Mass2Motif annotations, or the genome mining produced no sub-cluster predictions — there is nothing to correlate.
- You only need BGC sub-cluster prediction itself (run iPRESTO alone); this skill is specifically the downstream metabologenomic linkage step.

## Inputs

- AntiSMASH-predicted BGCs from each strain in the collection (e.g. 5,927 BGCs from 146 Streptomyces/Salinispora strains, antiSMASH 3.0)
- iPRESTO sub-cluster motifs and sub-cluster clans (SCCs) queried per strain
- MS/MS-derived Mass2Motif presence/absence per strain (molecular networking linkouts)

## Outputs

- Table of correlation scores and permutation p-values for all Mass2Motif–sub-cluster motif pairs and Mass2Motif–SCC pairs
- List of significant pairs (e.g. 3,230 Mass2Motif–sub-cluster motif pairs; 1,939 Mass2Motif–SCC pairs)
- Highlighted top hits linking specific Mass2Motifs to sub-clusters (e.g. two staurosporine-related Mass2Motifs vs amino-sugar sub-cluster motifs/SCCs, Fig 6)

## How to apply

Query the BGCs mined by antiSMASH (here 5,927 BGCs from 146 Streptomyces/Salinispora strains) with iPRESTO against the antiSMASH-DB sub-cluster motifs and sub-cluster clans (SCCs). For each Mass2Motif–sub-cluster motif pair (107,590 pairs in this dataset) and each Mass2Motif–SCC pair (122,404 pairs), compute the Doroghazi et al. correlation score measuring how often the mass motif and sub-cluster co-occur across strains, under the assumption that a BGC is required for synthesis but may be cryptic. Because highly related strains share many BGCs and compounds and inflate co-occurrence, run permutation tests on all pairs to assess how likely a high correlation score is by chance. Assemble a table of correlation scores and permutation p-values, flagging the significant pairs (3,230 Mass2Motif–sub-cluster motif pairs and 1,939 Mass2Motif–SCC pairs were significant here).

## Related tools

- **iPRESTO** (Query each strain's BGCs against antiSMASH-DB sub-cluster motifs and sub-cluster clans (SCCs) to obtain the genome-side features for correlation) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Provides the BGC predictions per strain that are tokenised and queried with iPRESTO (antiSMASH 3.0 for the 146-strain dataset))
- **Mass2Motifs** (MS/MS-derived substructure patterns per strain that form the metabolomics-side features for correlation)

## Evaluation signals

- The output table reports 3,230 significant Mass2Motif–sub-cluster motif pairs and 1,939 significant Mass2Motif–SCC pairs after permutation testing.
- The five pairs between the two staurosporine-related Mass2Motifs and the amino-sugar sub-cluster motifs/SCCs show high correlation scores with low p-values (Fig 6).
- Permutation p-values are present for all tested pairs, and the test design accounts for inflation caused by highly related strains sharing many BGCs and compounds.
- Correlation scores are computed under the stated biological assumption that a BGC is required for synthesis but may be cryptic.
- The pair counts tested match the dataset size: 107,590 Mass2Motif–sub-cluster motif pairs and 122,404 Mass2Motif–SCC pairs.

## Limitations

- Co-occurrence is inflated by highly related strains sharing many BGCs and compounds, so permutation tests are essential to control for this inflation.
- Numbers of antiSMASH or iPRESTO versions are not stated in the materials here; reproducibility of BGC mining depends on tool versions.
- The correlation assumes a BGC is required for synthesis but may be cryptic, which can leave mass motifs without matching sub-clusters unmatched.

## Evidence

- [abstract] Metabologenomic linkage via co-occurrence correlation: "for a collection of 145 actinobacteria how substructures can aid in linking BGCs to molecules by correlating iPRESTO-detected sub-clusters to MS/MS-derived Mass2Motifs substructure patterns"
- [intro] MS-guided genome mining motivation: "Chemical moieties identified from fragments in mass spectrometry (MS) data could be linked to sub-clusters responsible for their synthesis, as part of MS-guided genome mining strategies"
- [supplementary] Correlation scores are the core output: "Correlation scores between Mass2Motifs and sub-clusters"
- [abstract] Sub-clusters detected with iPRESTO are the genome-side features: "iPRESTO was used to mine sub-clusters across 150,000 prokaryotic BGCs from antiSMASH-DB"
