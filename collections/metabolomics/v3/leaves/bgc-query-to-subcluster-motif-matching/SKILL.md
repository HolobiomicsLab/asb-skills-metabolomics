---
name: bgc-query-to-subcluster-motif-matching
description: Use when tokenised (sub)Pfam-domain BGCs must be matched against pre-trained
  LDA sub-cluster motifs to predict biosynthetic sub-clusters, or to validate predicted
  sub-clusters against known SubClusterBlast entries.
when_to_use_negative:
- When gene order matters for the hypothesis being tested — the LDA branch assumes
  a bag-of-words model where original gene order is not considered; use the PRESTO-STAT
  co-occurrence approach instead, which uses tokens in their original order
- When no trained motif model exists and de novo sub-cluster discovery is the goal
  — this skill queries existing motifs, it does not train them
- When sub-clusters consist of a single gene — such matches are treated as noise and
  are not counted
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0085
tools:
- name: iPRESTO
  role: Provides the PRESTO-TOP LDA sub-cluster prediction and querying functionality
    used to match BGCs to motifs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Source of the predicted BGCs that are tokenised into (sub)Pfam domain combinations
    prior to querying
- name: SubClusterBlast
  role: Reference module of experimentally verified sub-clusters used to validate
    query results at the 0.6 overlap cut-off
- name: build_subpfam
  role: Builds the per-clade subPfam pHMMs used to tokenise BGC genes into finer-grained
    domains before querying
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:e33e538e7de5b440f1926cb5eaea33dff8ffe957302358aa3aea0a7e8517481f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# bgc-query-to-subcluster-motif-matching

## Summary

Query tokenised biosynthetic gene clusters (BGCs) against a trained PRESTO-TOP LDA sub-cluster motif model to predict which biosynthetic sub-clusters (e.g. methoxymalonate, AHBA) are present, enabling substructure-level annotation of BGCs.

## When to use

Use when tokenised (sub)Pfam-domain BGCs must be matched against pre-trained LDA sub-cluster motifs to predict biosynthetic sub-clusters, or to validate predicted sub-clusters against known SubClusterBlast entries.

## When NOT to use

- When gene order matters for the hypothesis being tested — the LDA branch assumes a bag-of-words model where original gene order is not considered; use the PRESTO-STAT co-occurrence approach instead, which uses tokens in their original order
- When no trained motif model exists and de novo sub-cluster discovery is the goal — this skill queries existing motifs, it does not train them
- When sub-clusters consist of a single gene — such matches are treated as noise and are not counted

## Inputs

- Tokenised BGCs represented as bag-of-words frequency vectors of (sub)Pfam domain combinations (non-biosynthetic Pfams removed)
- Trained PRESTO-TOP LDA sub-cluster motif model (e.g. 1,000 topics trained on antiSMASH-DB)
- Optional reference set of experimentally verified sub-clusters from the antiSMASH SubClusterBlast module for validation

## Outputs

- Predicted sub-cluster assignments per BGC (gene-to-motif probability assignments passing the 0.3 / 1.1 / 0.15 cut-offs)
- Validation overlap statistics vs. SubClusterBlast sub-clusters at the 0.6 overlap cut-off (e.g. 91 of 109 recovered, 83%)
- Matched sub-cluster motif labels (e.g. methoxymalonate and AHBA sub-clusters of macbecin)

## How to apply

Represent each BGC as a bag-of-words frequency vector of its (sub)Pfam domain combinations, ignoring gene order, as expected by the LDA (topic-modelling) branch of iPRESTO. Query the BGC against the trained motif set (e.g. 1,000 topics trained on the non-redundant 60,028-BGC antiSMASH-DB dataset) and accept a motif match when each gene has a gene-to-motif probability above 0.3, the summed gene-to-motif probabilities exceed 1.1 (implying more than one gene), and the overlap score of summed present domain-combination probabilities is at least 0.15. For validation against reference sub-clusters (e.g. the 109 experimentally verified antiSMASH SubClusterBlast sub-clusters), count a hit when at least 0.6 of the SubClusterBlast sub-cluster's genes are found in a predicted sub-cluster. To reduce noise in the motif definitions themselves, sort each motif's domain-combination probability vector and retain combinations until a cumulative sum of 0.95 is reached.

## Related tools

- **iPRESTO** (Provides the PRESTO-TOP LDA sub-cluster prediction and querying functionality used to match BGCs to motifs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Source of the predicted BGCs that are tokenised into (sub)Pfam domain combinations prior to querying)
- **SubClusterBlast** (Reference module of experimentally verified sub-clusters used to validate query results at the 0.6 overlap cut-off)
- **build_subpfam** (Builds the per-clade subPfam pHMMs used to tokenise BGC genes into finer-grained domains before querying)

## Evaluation signals

- Every accepted motif match satisfies all three cut-offs: per-gene gene-to-motif probability > 0.3, summed gene-to-motif probabilities > 1.1, and overlap score >= 0.15
- Against the 109 SubClusterBlast reference sub-clusters at the 0.6 overlap cut-off, the 1,000-topic model recovers 91 (83%); substantially lower recovery indicates a training or query-parameter problem
- Log-likelihood on held-out BGCs converges and u_mass coherence scores are computed as model-quality checks before querying
- Motif domain-combination probability vectors are truncated at cumulative sum 0.95 — retained combinations beyond that point indicate the filtering step was skipped

## Limitations

- The LDA/bag-of-words representation ignores gene order, so sub-clusters whose function depends on synteny may be mis-modelled
- Matches of length 1 (single-gene sub-clusters) are ignored as almost definite noise
- No version numbers for antiSMASH or iPRESTO are stated in the supplementary material, and reproducibility depends on the exact Gensim LDA settings (symmetric alpha and beta = 1/N, 500 iterations, mini-batch chunk of 5% of data with minimum 2,000)

## Evidence

- [abstract] Scalable unsupervised sub-cluster detection via iPRESTO: "we constructed a scalable method for unsupervised sub-cluster detection, called iPRESTO"
- [intro] iPRESTO uses PRESTO-STAT and PRESTO-TOP: "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
- [supplementary] LDA bag-of-words assumption: "or by LDA, which assumes a bag of words model where original gene order is not considered"
- [supplementary] 0.6 overlap cut-off for sub-cluster validation: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] Tokenisation of antiSMASH BGCs into (sub)Pfams: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
