---
name: subcluster-motif-probability-filtering
description: Use when querying tokenised BGCs (bags of (sub)Pfam domain combinations)
  against trained LDA sub-cluster motifs and you must decide which gene-to-motif assignments
  count as real sub-cluster predictions versus noise.
when_to_use_negative:
- When gene order matters — LDA assumes a bag-of-words model where original gene order
  is not considered; use the PRESTO-STAT statistical method which keeps tokenised
  genes in their original order
- When the goal is de novo BGC discovery rather than sub-cluster prediction — the
  motifs only cover sub-clusters (e.g. methoxymalonate, AHBA motifs), not full BGC
  core machinery
- When the motif bank has not been trained — the thresholds are meaningless without
  per-motif and per-gene probability vectors from a fitted LDA model
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: iPRESTO
  role: Implements the PRESTO-TOP LDA sub-cluster prediction and the probability-threshold
    filtering of motif queries
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Predicts the BGCs that are tokenised into (sub)Pfam domain combinations prior
    to LDA querying
- name: SubClusterBlast
  role: Provides the 109 experimentally verified sub-clusters used to validate filtered
    predictions at the 0.6 overlap cut-off
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:86ffce8b12fc768312efe85f0d87fe794f76e61dc17a7dc83f9c64af407892ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# subcluster-motif-probability-filtering

## Summary

Filters PRESTO-TOP LDA sub-cluster motifs and query hits using probability thresholds: motif domain-combinations are truncated at a cumulative probability of 0.95 to remove noise, and BGC gene-to-motif matches must pass gene-level (p > 0.3), summed (Σp > 1.1), and overlap (≥ 0.15) cut-offs before a sub-cluster prediction is accepted.

## When to use

Use when querying tokenised BGCs (bags of (sub)Pfam domain combinations) against trained LDA sub-cluster motifs and you must decide which gene-to-motif assignments count as real sub-cluster predictions versus noise.

## When NOT to use

- When gene order matters — LDA assumes a bag-of-words model where original gene order is not considered; use the PRESTO-STAT statistical method which keeps tokenised genes in their original order
- When the goal is de novo BGC discovery rather than sub-cluster prediction — the motifs only cover sub-clusters (e.g. methoxymalonate, AHBA motifs), not full BGC core machinery
- When the motif bank has not been trained — the thresholds are meaningless without per-motif and per-gene probability vectors from a fitted LDA model

## Inputs

- Trained PRESTO-TOP LDA motif bank (1,000 topics) with per-motif domain-combination probability vectors
- Tokenised BGCs represented as bag-of-words frequency vectors of (sub)Pfam domain combinations (non-biosynthetic Pfams removed)
- Optional: experimentally verified reference sub-clusters (e.g. antiSMASH SubClusterBlast sub-clusters) for validation

## Outputs

- Noise-filtered sub-cluster motifs (domain combinations up to cumulative probability 0.95)
- Accepted BGC-to-motif sub-cluster predictions meeting the 0.3 / 1.1 / 0.15 probability cut-offs
- Validation overlap counts at the 0.6 gene-overlap cut-off (e.g. 91 of 109 SubClusterBlast sub-clusters recovered)

## How to apply

After training the PRESTO-TOP LDA model (1,000 topics, Gensim multicore with online variational Bayes, symmetric alpha and beta = 1/N), first filter each motif: sort its domain-combination probability vector descending and retain combinations until the cumulative sum reaches 0.95, discarding the tail as noise. Then, when querying a BGC to the motif bank, accept a motif match only when (a) every gene in the candidate sub-cluster has a gene-to-motif probability above 0.3, (b) the summed gene-to-motif probabilities exceed 1.1 — enforcing that more than one gene participates — and (c) the overlap score of summed present domain-combination probabilities is at least 0.15. These thresholds operationalise the paper's principle that sub-clusters of length one are 'almost definitely noise' and that multi-gene probabilistic support is required. For validation against known sub-clusters (e.g. antiSMASH SubClusterBlast), count a hit when at least 0.6 of the reference sub-cluster's genes are recovered in a predicted sub-cluster.

## Related tools

- **iPRESTO** (Implements the PRESTO-TOP LDA sub-cluster prediction and the probability-threshold filtering of motif queries) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Predicts the BGCs that are tokenised into (sub)Pfam domain combinations prior to LDA querying)
- **SubClusterBlast** (Provides the 109 experimentally verified sub-clusters used to validate filtered predictions at the 0.6 overlap cut-off)

## Examples

```
from gensim.models import LdaMulticore; model = LdaMulticore(corpus, num_topics=1000, alpha='symmetric', eta='symmetric', passes=500)
```

## Evaluation signals

- Each retained motif's domain-combination probabilities sum to exactly 0.95 after truncation
- Every accepted sub-cluster prediction contains more than one gene (summed gene-to-motif probabilities > 1.1) and no gene below probability 0.3
- Overlap scores of accepted matches are ≥ 0.15; sub-clusters of length one are rejected as noise
- Against the 109 SubClusterBlast reference sub-clusters, the filtered model should recover ~91 (83%) at the 0.6 overlap cut-off
- Model quality monitored via u_mass coherence score and log-likelihood convergence on held-out BGCs

## Limitations

- Thresholds (0.3, 1.1, 0.15, 0.6) were tuned for the antiSMASH-DB dataset (60,028 non-redundant BGCs); they may not transfer directly to other BGC corpora
- LDA ignores gene order, so sub-cluster predictions cannot exploit adjacency information (unlike PRESTO-STAT's graph-based filtering with adjacency index ≥ 0.95)
- Matches of length 1 are ignored, so genuinely single-gene sub-clusters cannot be recovered

## Evidence

- [methods] Motif domain-combination truncation at 0.95 cumulative probability: [evidence span withheld — anchor status: not_found]
- [methods] Acceptance thresholds for BGC-to-motif queries: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation overlap cut-off: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] Length-one sub-clusters treated as noise: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [supplementary] LDA bag-of-words assumption: "or by LDA, which assumes a bag of words model where original gene order is not considered"
