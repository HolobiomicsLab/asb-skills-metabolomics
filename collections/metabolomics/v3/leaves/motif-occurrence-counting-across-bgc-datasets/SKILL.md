---
name: motif-occurrence-counting-across-bgc-datasets
description: Use when you have a trained sub-cluster motif library (e.g. PRESTO-TOP
  motifs annotated with substructures) and a large BGC dataset, and need per-motif
  occurrence counts, degree statistics, and BGC class distributions.
when_to_use_negative:
- You need to discover new sub-cluster motifs rather than quantify occurrences of
  an existing annotated library (use iPRESTO's PRESTO-TOP/PRESTO-STAT prediction instead).
- Length-1 sub-cluster matches are biologically meaningful in your context — this
  skill deliberately ignores them as noise.
- Your BGC dataset is not tokenised into (sub)Pfam terms; BGCs must first be tokenised
  by detecting (sub)Pfams per gene with non-biosynthetic Pfams removed.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Provides the PRESTO-TOP matching machinery and probability cut-offs used to
    find motif occurrences in queried BGCs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Source of the BGC predictions (antiSMASH-DB) in which motif occurrences are
    counted
- name: SubClusterBlast
  role: Supplies the 90 experimentally validated sub-clusters used to annotate motifs
    with substructures before counting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:aabaeec19b401891355ad48df583bc467df140c5342a3561711759cfd748fe7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SubClusterBlast
  - antiSMASH
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Motif occurrence counting across BGC datasets

## Summary

Counts occurrences of annotated PRESTO-TOP sub-cluster motifs (e.g. the 45 motifs covering 24 substructure types) across a non-redundant BGC dataset such as antiSMASH-DB, to quantify how often each encoded substructure appears and report per-motif degrees and BGC class distributions.

## When to use

Use when you have a trained sub-cluster motif library (e.g. PRESTO-TOP motifs annotated with substructures) and a large BGC dataset, and need per-motif occurrence counts, degree statistics, and BGC class distributions.

## When NOT to use

- You need to discover new sub-cluster motifs rather than quantify occurrences of an existing annotated library (use iPRESTO's PRESTO-TOP/PRESTO-STAT prediction instead).
- Length-1 sub-cluster matches are biologically meaningful in your context — this skill deliberately ignores them as noise.
- Your BGC dataset is not tokenised into (sub)Pfam terms; BGCs must first be tokenised by detecting (sub)Pfams per gene with non-biosynthetic Pfams removed.

## Inputs

- Trained PRESTO-TOP model with sub-cluster motifs (e.g. 1,000 motifs), annotated with substructures (45 motifs, 24 substructure types)
- Non-redundant antiSMASH-DB dataset of 60,028 tokenised BGCs
- MIBiG v1.4 reference dataset (1,819 BGCs) and the 90 experimentally validated SubClusterBlast sub-clusters used for motif annotation

## Outputs

- Per-motif occurrence counts (number of BGCs containing each motif, e.g. 19 for motif 190 to 873 for motif 220)
- Per-motif degree and BGC class distribution table
- Overall coverage statistic (e.g. ~9,500 / 16% of antiSMASH-DB BGCs containing at least one annotated motif)

## How to apply

Query the target BGC dataset (e.g. the non-redundant 60,028 BGCs from antiSMASH-DB) against the annotated motif library using the PRESTO-TOP matching cut-offs: gene-to-motif probability cut-off 0.3, summed gene-to-motif probability cut-off 1.1, and overlap score threshold 0.15 with the motif probability vector truncated at cumulative 0.95. When counting occurrences, ignore matches of length 1 because sub-clusters of length one are almost definitely noise, and count hybrid-class BGCs for all classes they contain. For each of the 45 annotated motifs, record the number of BGCs containing at least one match and the distribution over BGC product classes. In the reference study this produced occurrences ranging from 19 BGCs (motif 190) to 873 BGCs (motif 220), with roughly 9,500 (16%) of antiSMASH-DB BGCs containing at least one annotated motif.

## Related tools

- **iPRESTO** (Provides the PRESTO-TOP matching machinery and probability cut-offs used to find motif occurrences in queried BGCs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Source of the BGC predictions (antiSMASH-DB) in which motif occurrences are counted)
- **SubClusterBlast** (Supplies the 90 experimentally validated sub-clusters used to annotate motifs with substructures before counting)

## Examples

```
python query_bgc.py --query antiSMASH-DB_nr --model presto_top_motifs.hmm --gene_cutoff 0.3 --sum_cutoff 1.1 --overlap 0.15 --ignore_length_1
```

## Evaluation signals

- The annotated motif library contains exactly 45 motif rows spanning 24 distinct substructure types.
- Occurrence counts ignore length-1 matches; recomputing with them included should change results only via single-gene hits.
- Hybrid-class BGCs are counted once for each class they contain, so per-class counts may sum to more than the total BGC count.
- Per-motif occurrence counts fall within the reported range of 19 (motif 190) to 873 (motif 220) BGCs when run on the same non-redundant antiSMASH-DB dataset.
- Approximately 16% (~9,500) of the 60,028 non-redundant antiSMASH-DB BGCs contain at least one annotated motif.

## Limitations

- Matches of length 1 are almost certainly noise and are excluded, so motifs detectable only via single-gene hits will be undercounted.
- Coverage is partial: even with 45 annotated motifs, only ~16% of antiSMASH-DB BGCs get a substructure prediction.
- Annotation of motifs depends on matching quality against only 90 characterised SubClusterBlast sub-clusters and MIBiG BGC instances, so some motifs remain unannotated.

## Evidence

- [supplementary] Count occurrences of the 45 annotated motifs in the non-redundant antiSMASH-DB dataset, ignoring length-1 matches, and report per-motif degrees and BGC class distributions.: [evidence span withheld — anchor status: not_found]
- [supplementary] Matching cut-offs for querying BGCs against motifs.: [evidence span withheld — anchor status: not_found]
- [supplementary] Rationale for ignoring length-1 matches.: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [abstract] Overall coverage finding.: "we could predict a substructure for 16% of the antiSMASH-DB BGCs"
- [supplementary] Handling of hybrid-class BGCs.: "Matches of length 1 are ignored and hybrid class BGCs are counted for all classes they contain"
