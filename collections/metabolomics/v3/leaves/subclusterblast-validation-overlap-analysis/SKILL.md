---
name: subclusterblast-validation-overlap-analysis
description: Use when an unsupervised sub-cluster prediction model (e.g. PRESTO-TOP
  topics trained on tokenised BGCs) must be benchmarked against experimentally verified
  sub-clusters such as the 109 SubClusterBlast entries.
when_to_use_negative:
- 'When the goal is de novo sub-cluster discovery without a benchmark: the 0.6 overlap
  validation only quantifies recovery of known SubClusterBlast sub-clusters and cannot
  assess novelty'
- 'When gene order matters to the hypothesis: validation compares gene content overlaps,
  and the LDA bag-of-words representation ignores original gene order (unlike the
  PRESTO-STAT variant)'
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0157
tools:
- name: iPRESTO
  role: Command-line tool implementing PRESTO-TOP LDA sub-cluster prediction whose
    motifs are validated against SubClusterBlast
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Predicts the BGCs and provides the SubClusterBlast module holding the 109
    experimentally verified reference sub-clusters
- name: SubClusterBlast
  role: Source of experimentally verified sub-cluster annotations used as the validation
    reference set
- name: build_subpfam
  role: Builds per-clade subPfam pHMMs used to tokenise BGC genes before model training
    and validation
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:4da2ad80824bfc5dc4f83d97c61d0d578d43d8ec9a5ac152e635c8ca7ee0a225
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# subclusterblast-validation-overlap-analysis

## Summary

Validates predicted biosynthetic sub-clusters (e.g. PRESTO-TOP LDA motifs) against the experimentally verified sub-clusters stored in the antiSMASH SubClusterBlast module, counting a hit when at least 0.6 of the reference sub-cluster's genes are recovered in a predicted sub-cluster.

## When to use

Use when an unsupervised sub-cluster prediction model (e.g. PRESTO-TOP topics trained on tokenised BGCs) must be benchmarked against experimentally verified sub-clusters such as the 109 SubClusterBlast entries.

## When NOT to use

- When the goal is de novo sub-cluster discovery without a benchmark: the 0.6 overlap validation only quantifies recovery of known SubClusterBlast sub-clusters and cannot assess novelty
- When gene order matters to the hypothesis: validation compares gene content overlaps, and the LDA bag-of-words representation ignores original gene order (unlike the PRESTO-STAT variant)

## Inputs

- Non-redundant antiSMASH-DB dataset (60,028 BGCs) tokenised into (sub)Pfam domain-combination bag-of-words frequency vectors (gene order ignored)
- Trained PRESTO-TOP LDA model (e.g. 1,000 sub-cluster motifs / topics)
- antiSMASH SubClusterBlast module entries: 109 experimentally verified sub-clusters (including MIBiG 1.4 reference BGCs)

## Outputs

- Per-motif filtered domain-combination probability lists (retained until cumulative sum of 0.95)
- Sub-cluster match calls per BGC (gene-to-motif probability > 0.3, summed probabilities > 1.1, overlap score >= 0.15)
- Validation hit count / recovery rate against the 109 SubClusterBlast sub-clusters at the 0.6 overlap cut-off (e.g. 91/109 = 83%)

## How to apply

Train the candidate model on the non-redundant antiSMASH-DB dataset (60,028 BGCs tokenised into (sub)Pfam domain-combination bags-of-words; e.g. a 1,000-topic Gensim multicore LDA with symmetric alpha=beta=1/N, 500 iterations, mini-batch of max(5%, 2,000)). For each motif, sort domain-combination probabilities descending and retain those whose cumulative sum reaches 0.95 to filter noise. Query all BGCs to the motifs with match criteria of gene-to-motif probability > 0.3, summed gene-to-motif probabilities > 1.1 (implying more than one gene), and overlap score of summed present domain-combination probabilities >= 0.15. Then compare predicted sub-clusters to the antiSMASH SubClusterBlast reference set, counting a hit when at least 0.6 of the SubClusterBlast sub-cluster's genes are found within a predicted sub-cluster — the paper considers an overlap of 0.6 sufficient. For the iPRESTO PRESTO-TOP model this yields 91 of 109 recovered sub-clusters (83%), including methoxymalonate and AHBA sub-clusters of macbecin.

## Related tools

- **iPRESTO** (Command-line tool implementing PRESTO-TOP LDA sub-cluster prediction whose motifs are validated against SubClusterBlast) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Predicts the BGCs and provides the SubClusterBlast module holding the 109 experimentally verified reference sub-clusters)
- **SubClusterBlast** (Source of experimentally verified sub-cluster annotations used as the validation reference set)
- **build_subpfam** (Builds per-clade subPfam pHMMs used to tokenise BGC genes before model training and validation) — satriaphd/build_subpfam

## Examples

```
python presto_top.py --input antiSMASH-DB_nr_60028.gbk --topics 1000 --alpha symmetric --beta symmetric --iterations 500 --evaluate SubClusterBlast --overlap-cutoff 0.6
```

## Evaluation signals

- The trained model must yield exactly 1,000 sub-cluster motifs before validation
- Expected recovery rate: 91 of 109 (83%) experimentally verified SubClusterBlast sub-clusters matched at the 0.6 overlap cut-off
- Known motifs such as the methoxymalonate and AHBA sub-clusters of macbecin must be recovered
- u_mass coherence score and log-likelihood convergence on held-out BGCs should be monitored to confirm the LDA model is fit before validation
- Predicted sub-clusters of length one must be excluded as noise; hybrid-class BGCs are counted for all classes they contain

## Limitations

- Validation counts a hit only when >= 0.6 of the SubClusterBlast sub-cluster's genes are recovered, so partial recoveries below this cut-off are missed
- Non-biosynthetic Pfams are removed during tokenisation; subPfams are only constructed for the 112 most frequent Pfam domains in antiSMASH-DB, so rarer domains are coarser
- LDA treats BGCs as bag-of-words and ignores gene order, which may miss order-dependent sub-cluster structure

## Evidence

- [results] 83% recovery rate: [evidence span withheld — anchor status: not_found]
- [supplementary] 0.6 overlap cut-off: "We considered an overlap of 0.6 sufficient for having predicted a sub-cluster (see Supplementary methods in S1 Text)"
- [supplementary] bag-of-words representation: "or by LDA, which assumes a bag of words model where original gene order is not considered"
- [supplementary] BGC tokenisation: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [abstract] MIBiG confirmation: "our method was able to confirm 83% of the experimentally characterised sub-clusters in MIBiG reference BGCs"
