---
name: subcluster-motif-querying
description: Use when you have antiSMASH-predicted BGCs tokenised with (sub)Pfam domains
  and need to identify which PRESTO-TOP sub-cluster motifs they match, e.g. to nominate
  a candidate BGC for an orphan metabolite.
when_to_use_negative:
- The BGCs have not yet been tokenised into (sub)Pfam domain tokens — run antiSMASH
  and the tokenisation step first
- The goal is de novo sub-cluster discovery rather than querying existing motifs —
  use iPRESTO's PRESTO-STAT/PRESTO-TOP prediction mode instead
- You need to link metabolomics Mass2Motifs to sub-clusters across a strain panel
  — that requires the metabologenomic correlation workflow, not motif querying
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iPRESTO
  role: Runs the PRESTO-TOP query of tokenised BGCs against the annotated sub-cluster
    motif set with the 0.3/1.1/0.15 thresholds
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: antiSMASH
  role: Predicts the BGC regions and (sub)Pfam domain tokens that are subsequently
    queried against the motif set
- name: build_subpfam
  role: Builds the per-clade subPfam pHMM models used for fine-grained BGC tokenisation
    prior to motif querying
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:74c04a4551d82f1c8eb986030de913cad34e3ee12dc7675f6cde561e86ef83c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mass2Motifs
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# subcluster-motif-querying

## Summary

Query tokenised BGCs against a reference set of PRESTO sub-cluster motifs (e.g. the 1,000 annotated PRESTO-TOP motifs from antiSMASH-DB) to flag biosynthetic sub-structures — such as methylaminosugar or halogenated aromatic moieties — inside newly mined gene clusters, enabling genome-based structure prediction for orphan metabolites.

## When to use

Use when you have antiSMASH-predicted BGCs tokenised with (sub)Pfam domains and need to identify which PRESTO-TOP sub-cluster motifs they match, e.g. to nominate a candidate BGC for an orphan metabolite.

## When NOT to use

- The BGCs have not yet been tokenised into (sub)Pfam domain tokens — run antiSMASH and the tokenisation step first
- The goal is de novo sub-cluster discovery rather than querying existing motifs — use iPRESTO's PRESTO-STAT/PRESTO-TOP prediction mode instead
- You need to link metabolomics Mass2Motifs to sub-clusters across a strain panel — that requires the metabologenomic correlation workflow, not motif querying

## Inputs

- antiSMASH-predicted BGC regions with region identifiers (e.g. QZWF01000007.1.region003) from a genome assembly
- Tokenised BGC files: per-gene (sub)Pfam domain assignments from HMMER3 hmmscan with non-biosynthetic Pfams removed
- Annotated PRESTO-TOP sub-cluster motif set (S1 File; 1,000 motifs from antiSMASH-DB, 45 annotated via MIBiG)
- PRESTO-TOP threshold parameters (0.3 per-gene probability, 1.1 summed probability, 0.15 overlap score)

## Outputs

- Per-BGC-region motif-match table listing which sub-cluster motifs each region contains (e.g. akashin_bgc_subcluster_motifs.tsv)
- Motif annotations linking matched motifs to chemical sub-structures (methylaminosugar, halogenated aromatic ring, (amino)deoxysugar)
- Candidate BGC assignments for orphan metabolites (e.g. QZWF01000007.1.region003 as the akashin BGC)

## How to apply

First tokenise each antiSMASH-predicted BGC by detecting (sub)Pfam domains in every gene via HMMER3 hmmscan, removing non-biosynthetic Pfams so only biosynthetic tokens remain. Query the tokenised BGCs against the annotated motif set (e.g. the 1,000 PRESTO-TOP sub-cluster motifs mined from antiSMASH-DB) applying the PRESTO-TOP acceptance thresholds: 0.3 gene-to-motif probability per gene, 1.1 summed gene-to-motif probability, and 0.15 overlap score. Record which BGC regions contain which motifs, writing a per-region motif-match table. Annotate matched motifs where possible using the 45 sub-cluster motifs annotated from MIBiG reference BGCs (e.g. motif 194 = methylaminosugar, motif 607 = halogenated aromatic ring, motif 680 = (amino)deoxysugar). Validate by confirming that expected region identifiers and motifs are recovered — as in the akashin case, where QZWF01000007.1.region003 (StrepF001_25985–StrepF001_26130) was flagged by motifs 194, 607, and 680.

## Related tools

- **iPRESTO** (Runs the PRESTO-TOP query of tokenised BGCs against the annotated sub-cluster motif set with the 0.3/1.1/0.15 thresholds) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **antiSMASH** (Predicts the BGC regions and (sub)Pfam domain tokens that are subsequently queried against the motif set)
- **build_subpfam** (Builds the per-clade subPfam pHMM models used for fine-grained BGC tokenisation prior to motif querying) — satriaphd/build_subpfam

## Evaluation signals

- Output table contains region identifiers with matched motif IDs (e.g. QZWF01000007.1.region003 matched by motifs 194, 607, and 680 for akashin)
- All reported motif matches meet the PRESTO-TOP acceptance thresholds: gene-to-motif probability ≥ 0.3 per gene, summed probability ≥ 1.1, overlap score ≥ 0.15
- Matched motifs can be annotated to chemical sub-structures via the MIBiG-derived annotations (e.g. 194 = methylaminosugar)
- Querying a known reference BGC (e.g. rifamycin BGC0000373) recovers its expected sub-cluster motifs in both PRESTO-TOP and PRESTO-STAT outputs
- Candidate BGC assignments are consistent with independent evidence, e.g. the akashin region lying directly upstream of the diazaquinomycin BGC

## Limitations

- The annotated motif set is incomplete: only 45 of the 1,000 PRESTO-TOP motifs were annotated via MIBiG occurrences, so many matches carry no sub-structure label
- PRESTO-TOP's LDA assumes a bag-of-words model in which original gene order is not considered, so motif queries ignore spatial organisation within the cluster
- Only 16% of antiSMASH-DB BGCs could be assigned a predicted substructure, so absence of motif matches does not imply absence of sub-structures
- Motif matches flag candidate sub-structures but do not confirm BGC–metabolite links; metabologenomic validation via Mass2Motif correlation requires a paired genome–metabolome dataset

## Evidence

- [supplementary] The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [abstract] 16% substructure coverage on antiSMASH-DB: "we could predict a substructure for 16% of the antiSMASH-DB BGCs"
- [intro] iPRESTO command-line availability: "iPRESTO is available as a command-line tool at https://git.wageningenur.nl/bioinformatics/iPRESTO/."
- [intro] PRESTO-TOP vs PRESTO-STAT dual approach: "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
- [readme] build_subpfam README purpose: "Build per-clade pHMM models from a collection of
aligned protein sequences (in a multifasta format)."
