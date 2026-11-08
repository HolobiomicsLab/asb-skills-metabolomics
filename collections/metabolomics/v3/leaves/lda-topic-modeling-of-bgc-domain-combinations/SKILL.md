---
name: lda-topic-modeling-of-bgc-domain-combinations
description: Use when you have antiSMASH-predicted BGCs tokenised into (sub)Pfam domain
  combinations and need unsupervised discovery of sub-cluster motifs without relying
  on gene order.
when_to_use_negative:
- When gene order matters for the sub-cluster signal — the LDA (PRESTO-TOP) bag-of-words
  representation discards gene order; the statistical method (PRESTO-STAT) that preserves
  tokenised gene order is the appropriate alternative
- When you need per-clade functional resolution of highly frequent Pfam domains rather
  than domain-combination motifs — build per-clade subPfam pHMM models instead
- When BGCs have already been assigned to sub-clusters or a feature table of domain
  combinations exists — retraining LDA from scratch is unnecessary
tools:
- name: iPRESTO
  role: Command-line tool providing the PRESTO-TOP LDA topic-modelling sub-cluster
    prediction workflow
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Predicts the BGCs whose genes are tokenised by (sub)Pfam detection to build
    the bag-of-words corpus
- name: SubClusterBlast
  role: Source of 109 experimentally verified sub-clusters used as the validation
    reference set
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models used to refine (sub)Pfam tokenisation
    of BGC genes
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:c55cbcbf82887d08354ce667685f7f867786da4fe0bf5cefb7d56c6c56997cb9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lda-topic-modeling-of-bgc-domain-combinations

## Summary

Train an LDA topic model (PRESTO-TOP from iPRESTO) on bag-of-words vectors of (sub)Pfam domain combinations from BGCs to discover biosynthetic sub-cluster motifs, validated against experimentally verified SubClusterBlast sub-clusters.

## When to use

Use when you have antiSMASH-predicted BGCs tokenised into (sub)Pfam domain combinations and need unsupervised discovery of sub-cluster motifs without relying on gene order.

## When NOT to use

- When gene order matters for the sub-cluster signal — the LDA (PRESTO-TOP) bag-of-words representation discards gene order; the statistical method (PRESTO-STAT) that preserves tokenised gene order is the appropriate alternative
- When you need per-clade functional resolution of highly frequent Pfam domains rather than domain-combination motifs — build per-clade subPfam pHMM models instead
- When BGCs have already been assigned to sub-clusters or a feature table of domain combinations exists — retraining LDA from scratch is unnecessary

## Inputs

- Non-redundant antiSMASH-DB dataset of BGCs (60,028 BGCs tokenised into 10,539 domain combinations, including MIBiG 1.4 and Streptomyces/Salinispora BGCs)
- BGC tokenisation as (sub)Pfam domain combinations per gene, with non-biosynthetic Pfams removed
- Experimentally verified sub-clusters from the antiSMASH SubClusterBlast module (109 sub-clusters) for validation

## Outputs

- 1,000 sub-cluster motifs (LDA topics) with sorted domain-combination probability vectors filtered to cumulative sum 0.95
- Per-BGC gene-to-motif probability assignments and predicted sub-clusters passing the 0.3 / 1.1 / 0.15 match thresholds
- u_mass coherence score and log-likelihood convergence metrics on held-out BGCs
- Validation recovery rate against SubClusterBlast sub-clusters at the 0.6 overlap cut-off

## How to apply

Represent each BGC as a bag-of-words frequency vector of its (sub)Pfam domain combinations, deliberately ignoring gene order since LDA assumes a bag-of-words model. Train the PRESTO-TOP model with the Gensim multicore LDA implementation using online variational Bayes: 1,000 topics, symmetric alpha and beta set to 1/N, 500 iterations, and a mini-batch chunk size of 5% of the data (minimum 2,000). For each motif, sort the domain-combination probability vector descending and retain combinations until a cumulative sum of 0.95 is reached to filter out noise. Query BGCs to the motifs, accepting a match when each gene has gene-to-motif probability above 0.3, summed gene-to-motif probabilities exceed 1.1, and the overlap score of summed present domain-combination probabilities is at least 0.15. Assess quality with the u_mass coherence score and log-likelihood convergence on held-out BGCs, then validate against verified sub-clusters using a 0.6 overlap cut-off.

## Related tools

- **iPRESTO** (Command-line tool providing the PRESTO-TOP LDA topic-modelling sub-cluster prediction workflow) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Predicts the BGCs whose genes are tokenised by (sub)Pfam detection to build the bag-of-words corpus)
- **SubClusterBlast** (Source of 109 experimentally verified sub-clusters used as the validation reference set)
- **build_subpfam** (Builds per-clade subPfam pHMM models used to refine (sub)Pfam tokenisation of BGC genes) — satriaphd/build_subpfam

## Examples

```
LdaMulticore(corpus=bgc_bow, num_topics=1000, id2word=dictionary, alpha=1/1000, eta=1/1000, iterations=500, chunksize=2000)
```

## Evaluation signals

- The trained model yields exactly 1,000 sub-cluster motifs with domain-combination probability vectors truncated at cumulative sum 0.95
- Validation recovers 91 of 109 experimentally verified SubClusterBlast sub-clusters (83% recovery) at the 0.6 overlap cut-off
- Matched sub-clusters satisfy all three thresholds: gene-to-motif probability > 0.3, summed gene-to-motif probabilities > 1.1 (more than one gene), and overlap score >= 0.15
- u_mass coherence score computed and log-likelihood on held-out BGCs shows convergence
- Sub-cluster motifs of length one are treated as noise and not counted

## Limitations

- Non-biosynthetic Pfams must be removed during tokenisation; contamination degrades the bag-of-words signal
- An 83% recovery rate means ~17% of verified SubClusterBlast sub-clusters are not recovered at the 0.6 overlap cut-off
- Sub-clusters of length one are almost definitely noise and must be excluded
- No version numbers are stated for antiSMASH or iPRESTO in the supplementary material, complicating exact reproduction

## Evidence

- [supplementary] LDA bag-of-words assumption discards gene order: "or by LDA, which assumes a bag of words model where original gene order is not considered"
- [abstract] 83% recovery of verified sub-clusters: "our method was able to confirm 83% of the experimentally characterised sub-clusters in MIBiG reference BGCs"
- [supplementary] BGC tokenisation into (sub)Pfams: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene"
- [supplementary] 0.6 overlap cut-off for validation: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [abstract] Scalable unsupervised sub-cluster detection rationale: "we constructed a scalable method for unsupervised sub-cluster detection, called iPRESTO"
- [readme] subPfam construction from phylogenetic clades: [evidence span withheld — anchor status: external_doc]
