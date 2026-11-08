---
name: orphan-metabolite-bgc-linkage
description: Use when an orphan metabolite's known substructures (e.g. sugars, methylaminosugars,
  halogenated aromatics) must be matched to a candidate BGC region in a genome assembly
  that antiSMASH annotates but BGC databases lack.
when_to_use_negative:
- The target BGC is already experimentally characterised and present in MIBiG, so
  sub-cluster-based linkage adds nothing over direct reference lookup
- The goal is full BGC product class prediction rather than linking a specific metabolite
  substructure to a region via sub-cluster motifs
- The metabolite has no known substructures and no sub-cluster motifs can be defined
  for it, leaving no motifs to query against
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: antiSMASH
  role: Predicts BGC regions in the genome assembly that are subsequently tokenised
    and queried
- name: iPRESTO
  role: Tokenises BGC genes into (sub)Pfam domains and queries them to PRESTO-TOP
    sub-cluster motifs to flag candidate BGCs
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models used for finer-grained domain tokenisation
    of BGC genes
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:4d28aaea34e1bd3a8df0131d61a9d3a7bc9f1207cc5fadef95b39cac342c8b7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# orphan-metabolite-bgc-linkage

## Summary

Links orphan (or database-missing) metabolites to candidate biosynthetic gene clusters by tokenising antiSMASH-predicted BGCs into (sub)Pfam domains and querying them against PRESTO-TOP sub-cluster motifs, as demonstrated by proposing the akashin BGC QZWF01000007.1.region003 in Streptomyces sp. F001.

## When to use

Use when an orphan metabolite's known substructures (e.g. sugars, methylaminosugars, halogenated aromatics) must be matched to a candidate BGC region in a genome assembly that antiSMASH annotates but BGC databases lack.

## When NOT to use

- The target BGC is already experimentally characterised and present in MIBiG, so sub-cluster-based linkage adds nothing over direct reference lookup
- The goal is full BGC product class prediction rather than linking a specific metabolite substructure to a region via sub-cluster motifs
- The metabolite has no known substructures and no sub-cluster motifs can be defined for it, leaving no motifs to query against

## Inputs

- Whole-genome shotgun assembly containing the target contigs (e.g. contig QZWF01000007 from Streptomyces sp. F001)
- Annotated PRESTO-TOP sub-cluster motif set (S1 File, 1,000 motifs with functional annotations such as 194 = methylaminosugar)
- antiSMASH-predicted BGC regions with gene identifiers (e.g. StrepF001_25985–StrepF001_26130)
- (Optional) MS/MS-derived Mass2Motifs for metabologenomic correlation

## Outputs

- Per-region sub-cluster motif-match table (e.g. akashin_bgc_subcluster_motifs.tsv) listing matched motif IDs and their substructure annotations per BGC region
- Candidate BGC region assignment for the orphan metabolite (e.g. QZWF01000007.1.region003 as the akashin BGC, upstream of the diazaquinomycin BGC)

## How to apply

Run antiSMASH on the whole-genome shotgun assembly to detect BGC regions (e.g. QZWF01000007.1.region003), then tokenise every gene in each predicted region by detecting (sub)Pfam domains via HMMER3 hmmscan, removing non-biosynthetic Pfams. Query the tokenised BGCs against the annotated set of 1,000 PRESTO-TOP sub-cluster motifs, applying the PRESTO-TOP thresholds of 0.3 gene-to-motif probability per gene, 1.1 summed gene-to-motif probability, and 0.15 overlap score. Record which regions match which annotated motifs (e.g. motif 194 methylaminosugar, 607 halogenated aromatic ring, 680 (amino)deoxysugar) and flag the region hit by the motifs corresponding to the target metabolite's substructures. Where MS/MS data exist, correlate iPRESTO sub-clusters to Mass2Motifs to strengthen the metabolite–BGC link; ignore sub-clusters of length one as noise. The rationale is that sub-clusters encode substructures, so motif co-occurrence in an unclassified region predicts which orphan compound it makes.

## Related tools

- **antiSMASH** (Predicts BGC regions in the genome assembly that are subsequently tokenised and queried)
- **iPRESTO** (Tokenises BGC genes into (sub)Pfam domains and queries them to PRESTO-TOP sub-cluster motifs to flag candidate BGCs) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **build_subpfam** (Builds per-clade subPfam pHMM models used for finer-grained domain tokenisation of BGC genes) — satriaphd/build_subpfam

## Evaluation signals

- The motif-match table must show the target region matched by the expected motifs — for the akashin case, QZWF01000007.1.region003 (StrepF001_25985–StrepF001_26130) matched by motifs 194, 607, and 680
- PRESTO-TOP query thresholds must be respected: ≥0.3 gene-to-motif probability per gene, ≥1.1 summed gene-to-motif probability, and ≥0.15 overlap score
- Tokenisation output must exclude non-biosynthetic Pfams before querying motifs
- Sub-cluster predictions of length one must be discarded as noise
- Sanity check against the paper's benchmark: the same approach confirmed 83% of experimentally characterised sub-clusters in MIBiG reference BGCs and correctly recovered the xenorhabdin and salbostatin BGCs

## Limitations

- The article states no version numbers for antiSMASH or iPRESTO in the supplementary description, so exact tool versions must be fixed separately for reproducibility
- Sub-clusters of length one are almost certainly noise and cannot be used as evidence for a linkage
- Only ~16% of antiSMASH-DB BGCs could be assigned a predicted substructure, so many BGCs will lack informative motif hits

## Evidence

- [abstract] correctly identify the BGCs for xenorhabdin and salbostatin biosynthesis (which had not yet been annotated in BGC databases), as well as propose a candidate BGC for akashin biosynthesis: "correctly identify the BGCs for xenorhabdin and salbostatin biosynthesis (which had not yet been annotated in BGC databases), as well as propose a candidate BGC for akashin biosynthesis"
- [supplementary] Tokenisation removes non-biosynthetic Pfams before sub-cluster prediction: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [intro] iPRESTO is a command-line tool: "iPRESTO is available as a command-line tool at https://git.wageningenur.nl/bioinformatics/iPRESTO/."
- [abstract] Sub-cluster mining scale across antiSMASH-DB: "iPRESTO was used to mine sub-clusters across 150,000 prokaryotic BGCs from antiSMASH-DB"
- [readme] subPfam construction rationale from README: [evidence span withheld — anchor status: external_doc]
