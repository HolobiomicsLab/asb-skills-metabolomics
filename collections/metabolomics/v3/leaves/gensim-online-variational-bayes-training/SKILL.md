---
name: gensim-online-variational-bayes-training
description: Use when you need to learn sub-cluster motifs from tokenised BGCs — each
  BGC represented as a bag of (sub)Pfam domain combinations — via LDA topic modelling,
  e.g. to mine antiSMASH-DB scale data for biosynthetic sub-clusters.
when_to_use_negative:
- When gene order within BGCs is biologically important — the LDA bag-of-words model
  discards it; the complementary PRESTO-STAT statistical method preserves original
  gene order
- When input BGCs have not been tokenised into (sub)Pfam domain combinations — raw
  GenBank/antiSMASH output must first be converted to bag-of-words vectors
- When sub-clusters of length one are of interest — these are almost definitely noise
  and are not counted
edam_operation: http://edamontology.org/operation_0291
tools:
- name: iPRESTO
  role: Command-line tool implementing PRESTO-TOP LDA sub-cluster motif training and
    querying
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: Gensim
  role: Provides the multicore LDA implementation with online variational Bayes used
    to train the 1,000-topic model
- name: antiSMASH
  role: Supplies the BGC predictions that are tokenised into (sub)Pfam domain combinations
    as LDA input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:6479211959e4f078d797a5c22e04a139545dc322ee2044657d44a4881f05c784
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gensim
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0291
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# Gensim online variational Bayes LDA training

## Summary

Trains a topic model (PRESTO-TOP) over bag-of-words (sub)Pfam domain-combination vectors of BGCs using Gensim's multicore LDA with online variational Bayes, learning 1,000 sub-cluster motifs that encode recurring biosynthetic gene combinations independent of gene order.

## When to use

Use when you need to learn sub-cluster motifs from tokenised BGCs — each BGC represented as a bag of (sub)Pfam domain combinations — via LDA topic modelling, e.g. to mine antiSMASH-DB scale data for biosynthetic sub-clusters.

## When NOT to use

- When gene order within BGCs is biologically important — the LDA bag-of-words model discards it; the complementary PRESTO-STAT statistical method preserves original gene order
- When input BGCs have not been tokenised into (sub)Pfam domain combinations — raw GenBank/antiSMASH output must first be converted to bag-of-words vectors
- When sub-clusters of length one are of interest — these are almost definitely noise and are not counted

## Inputs

- Non-redundant antiSMASH-DB dataset (60,028 BGCs, incl. MIBiG 1.4 and Streptomyces/Salinispora BGCs)
- Per-BGC bag-of-words frequency vectors of 10,539 (sub)Pfam domain combinations (tokens)
- Held-out BGCs for log-likelihood convergence monitoring
- 109 experimentally verified SubClusterBlast sub-clusters for validation

## Outputs

- 1,000 sub-cluster motifs (LDA topics) as domain-combination probability vectors filtered to cumulative probability 0.95
- Per-BGC gene-to-motif probability assignments (matches: gene prob > 0.3, summed probs > 1.1, overlap score >= 0.15)
- u_mass coherence score and held-out log-likelihood convergence curves

## How to apply

Tokenise each BGC into (sub)Pfam domain combinations (removing non-biosynthetic Pfams) and represent it as a bag-of-words frequency vector, ignoring gene order because LDA assumes a bag-of-words model. Train PRESTO-TOP with Gensim's LdaMulticore using online variational Bayes: number of topics (sub-cluster motifs) set to 1,000, symmetric alpha and beta of 1/N, 500 iterations, and a mini-batch chunk size of 5% of the data with a minimum of 2,000 documents. For each learned motif, sort the domain-combination probability vector from high to low probability and retain domain combinations until a cumulative sum of 0.95 is reached to filter out noise. Assess model quality with the u_mass coherence score and by monitoring log-likelihood convergence on held-out BGCs; query BGCs against the motifs accepting matches when each gene's gene-to-motif probability exceeds 0.3, summed probabilities exceed 1.1, and the overlap score is at least 0.15.

## Related tools

- **iPRESTO** (Command-line tool implementing PRESTO-TOP LDA sub-cluster motif training and querying) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **Gensim** (Provides the multicore LDA implementation with online variational Bayes used to train the 1,000-topic model)
- **antiSMASH** (Supplies the BGC predictions that are tokenised into (sub)Pfam domain combinations as LDA input)

## Examples

```
from gensim.models import LdaMulticore; lda = LdaMulticore(corpus=bgc_bow, num_topics=1000, alpha=1/1000, eta=1/1000, iterations=500, chunksize=max(2000, int(0.05*len(bgc_bow))), passes=1)
```

## Evaluation signals

- Model yields exactly 1,000 sub-cluster motifs (topics) after training
- Held-out log-likelihood converges and u_mass coherence score is computed as a model-quality check
- Recovered motifs match 91 of 109 (83%) experimentally verified SubClusterBlast sub-clusters at the 0.6 overlap cut-off
- Motif domain-combination lists are truncated at a cumulative probability sum of 0.95
- Querying known BGCs (e.g. rifamycin BGC0000373, macbecin methoxymalonate/AHBA sub-clusters) reproduces expected motif hits

## Limitations

- LDA assumes a bag-of-words model, so original gene order is not considered — spatial/adjacency information is lost
- Matches of length 1 are ignored, and sub-clusters of length one are treated as almost definite noise
- No version numbers are stated for antiSMASH or iPRESTO in the supplementary methods, complicating exact reproduction
- Validation recovery was 83% (91/109), so roughly one in six verified sub-clusters is not recovered at the 0.6 overlap cut-off

## Evidence

- [methods] Train the PRESTO-TOP model with the Gensim multicore LDA implementation using online variational Bayes, setting the number of sub-cluster motifs (topics) to 1,000: [evidence span withheld — anchor status: not_found]
- [methods] coherence and convergence monitoring: [evidence span withheld — anchor status: not_found]
- [methods] query acceptance thresholds: [evidence span withheld — anchor status: not_found]
- [supplementary] bag-of-words assumption: "or by LDA, which assumes a bag of words model where original gene order is not considered"
- [supplementary] validation cut-off: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] input dataset scale: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
