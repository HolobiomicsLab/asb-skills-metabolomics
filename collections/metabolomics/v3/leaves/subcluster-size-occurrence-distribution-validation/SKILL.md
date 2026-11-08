---
name: subcluster-size-occurrence-distribution-validation
description: Use when a PRESTO-STAT run on tokenised antiSMASH-DB BGCs has produced
  a sub-cluster table and you must verify the detection count and the size and BGC-occurrence
  distributions reproduce the reported benchmarks.
when_to_use_negative:
- Sub-clusters were produced by the PRESTO-TOP LDA topic-modelling approach rather
  than the PRESTO-STAT statistical method — the 108,085 count and distribution benchmarks
  apply to the statistical method only
- The input dataset is not the filtered non-redundant antiSMASH-DB set (60,028 BGCs),
  since count and distribution expectations are dataset-specific
- Sub-clusters shorter than three genes or occurring in only one BGC have not yet
  been filtered out, making distribution metrics incomparable to the published benchmark
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0797
tools:
- name: iPRESTO
  role: Provides the PRESTO-STAT statistical sub-cluster detection whose output table
    is validated by this skill
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Supplies the prokaryotic BGC predictions from antiSMASH-DB that are tokenised
    and filtered before PRESTO-STAT runs
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models used to tokenise genes into (sub)Pfam
    domain combinations
  repo: https://github.com/satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:ff299f02735bbcdd4bec61fddc819247d3d1f9b969dc0f1e55b9878cad848730
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Sub-cluster size and occurrence distribution validation

## Summary

Validates output of iPRESTO's PRESTO-STAT statistical sub-cluster detection by checking the total count (108,085 sub-clusters on the filtered non-redundant antiSMASH-DB dataset of 60,028 BGCs) and the size/occurrence distributions (over 80% of sub-clusters contain fewer than ten genes; 17% occur in more than 10 BGCs).

## When to use

Use when a PRESTO-STAT run on tokenised antiSMASH-DB BGCs has produced a sub-cluster table and you must verify the detection count and the size and BGC-occurrence distributions reproduce the reported benchmarks.

## When NOT to use

- Sub-clusters were produced by the PRESTO-TOP LDA topic-modelling approach rather than the PRESTO-STAT statistical method — the 108,085 count and distribution benchmarks apply to the statistical method only
- The input dataset is not the filtered non-redundant antiSMASH-DB set (60,028 BGCs), since count and distribution expectations are dataset-specific
- Sub-clusters shorter than three genes or occurring in only one BGC have not yet been filtered out, making distribution metrics incomparable to the published benchmark

## Inputs

- presto_stat_subclusters.tsv — full table of PRESTO-STAT-detected sub-clusters
- Filtered, tokenised non-redundant antiSMASH-DB dataset (60,028 BGCs; 10,539 domain combinations; genes represented as (sub)Pfam domain combinations)

## Outputs

- Validated count of exactly 108,085 statistical sub-clusters
- Sub-cluster size distribution showing over 80% of sub-clusters with fewer than ten genes
- BGC-occurrence distribution showing 17% of sub-clusters occurring in more than 10 BGCs

## How to apply

First run PRESTO-STAT on the filtered, tokenised non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations), where each gene is a combination of (sub)Pfam domains, applying Benjamini–Yekutieli FDR correction and extracting maximal cliques below the 0.1 p-value threshold while filtering out sub-clusters with fewer than three genes or occurring in only one BGC. Then write the full result to presto_stat_subclusters.tsv and read it back to assert it contains exactly 108,085 sub-clusters. Compute the size distribution (number of genes per sub-cluster) and confirm that over 80% of statistical sub-clusters contain fewer than ten genes. Compute the occurrence distribution (number of distinct BGCs per sub-cluster) and confirm that 17% occur in more than 10 BGCs. Deviations in count or distribution shape indicate a mismatch in tokenisation, filtering, thresholding, or clique-extraction parameters relative to the reference configuration.

## Related tools

- **iPRESTO** (Provides the PRESTO-STAT statistical sub-cluster detection whose output table is validated by this skill) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Supplies the prokaryotic BGC predictions from antiSMASH-DB that are tokenised and filtered before PRESTO-STAT runs)
- **build_subpfam** (Builds per-clade subPfam pHMM models used to tokenise genes into (sub)Pfam domain combinations) — https://github.com/satriaphd/build_subpfam

## Evaluation signals

- presto_stat_subclusters.tsv contains exactly 108,085 rows/sub-clusters
- Over 80% of statistical sub-clusters contain fewer than ten genes
- 17% of sub-clusters occur in more than 10 BGCs
- No retained sub-cluster has fewer than three genes or occurs in only one BGC (filter was applied before distribution computation)
- Sub-cluster families/clans group consistently when K-means (scikit-learn) is applied to the presence/absence matrix of tokenised genes

## Limitations

- Sub-clusters of length one are not counted because they are almost definitely noise
- The benchmark count and distributions hold only for the filtered non-redundant antiSMASH-DB dataset (60,028 BGCs, 10,539 domain combinations) and the conservative larger-of-two p-value plus Benjamini–Yekutieli FDR configuration
- Hybrid class BGCs are counted for all classes they contain, and matches of length 1 are ignored, which affects occurrence counting

## Evidence

- [results] PRESTO-STAT detected 108,085 sub-clusters on the filtered dataset; over 80% of the statistical sub-clusters contain fewer than ten genes, and 17% occur in more than 10 BGCs.: [evidence span withheld — anchor status: not_found]
- [supplementary] not counting sub-clusters of length one as these are almost definitely noise (see Methods): "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [supplementary] the non-redundant 60,028 BGCs from the antiSMASH-DB dataset: "the non-redundant 60,028 BGCs from the antiSMASH-DB dataset"
- [intro] To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP).: "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
