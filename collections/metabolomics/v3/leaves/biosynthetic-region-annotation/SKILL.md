---
name: biosynthetic-region-annotation
description: Use when a genome assembly must be searched for candidate BGC regions
  responsible for known or orphan metabolites, and sub-cluster motifs (e.g. methylaminosugar,
  (amino)deoxysugar, halogenated aromatic ring) are needed to prioritise the region.
when_to_use_negative:
- The input is MS/MS data needing molecular substructure motifs (Mass2Motifs) rather
  than genomic regions — correlation of Mass2Motifs to sub-clusters is a separate
  downstream step, not region annotation itself
- The BGCs are already fully characterised and present in MIBiG — re-annotating known
  reference clusters adds little
- The goal is de novo sub-cluster discovery rather than querying existing regions
  against an annotated motif set
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3172
tools:
- name: antiSMASH
  role: Detects putative BGC regions in the genome assembly and provides region identifiers
    that are then tokenised
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: iPRESTO
  role: Tokenises BGC genes by (sub)Pfam detection and queries them against annotated
    PRESTO-TOP sub-cluster motifs to flag candidate regions
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models from aligned protein sequences used for
    finer-grained gene tokenisation
  repo: satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:c4573482b6b1e830ab3fd83d4b42e430d053786ed3e855ed7d8d2f2e84265265
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# biosynthetic-region-annotation

## Summary

Annotate candidate biosynthetic gene cluster (BGC) regions in a genome and flag the sub-cluster motifs (e.g. sugar, halogenated-ring modules) they contain, using antiSMASH region detection followed by iPRESTO sub-cluster motif querying. This links orphan metabolites to genomic loci via substructure-level motif signatures.

## When to use

Use when a genome assembly must be searched for candidate BGC regions responsible for known or orphan metabolites, and sub-cluster motifs (e.g. methylaminosugar, (amino)deoxysugar, halogenated aromatic ring) are needed to prioritise the region.

## When NOT to use

- The input is MS/MS data needing molecular substructure motifs (Mass2Motifs) rather than genomic regions — correlation of Mass2Motifs to sub-clusters is a separate downstream step, not region annotation itself
- The BGCs are already fully characterised and present in MIBiG — re-annotating known reference clusters adds little
- The goal is de novo sub-cluster discovery rather than querying existing regions against an annotated motif set

## Inputs

- Whole-genome shotgun assembly containing the target contig (e.g. contig QZWF01000007 from Streptomyces sp. F001)
- Annotated sub-cluster motif set (PRESTO-TOP motifs with annotations, e.g. the S1 File listing 1,000 motifs)
- (sub)Pfam HMM library for tokenisation via HMMER3 hmmscan

## Outputs

- antiSMASH region identifiers with gene ranges (e.g. QZWF01000007.1.region003, StrepF001_25985–StrepF001_26130)
- Per-region sub-cluster motif-match table (e.g. akashin_bgc_subcluster_motifs.tsv) listing matched motifs such as 194 (methylaminosugar), 607 (halogenated aromatic ring), 680 ((amino)deoxysugar)
- Candidate BGC assignment for the target metabolite

## How to apply

Run antiSMASH (version 6 in the source workflow) on the whole-genome shotgun assembly to detect putative BGCs and retain region identifiers (e.g. QZWF01000007.1.region003). Tokenise each gene in the predicted BGCs with iPRESTO by detecting (sub)Pfam domains via HMMER3 hmmscan, removing non-biosynthetic Pfams; subPfams are per-clade pHMMs built by splitting Pfam multiple sequence alignments into clades. Query the tokenised BGCs against the 1,000 PRESTO-TOP sub-cluster motifs using the PRESTO-TOP thresholds of 0.3 gene-to-motif probability per gene, 1.1 summed gene-to-motif probability, and 0.15 overlap score. Record which regions match which annotated motifs (e.g. motifs 194, 607, 680) and write a per-region motif-match table; a region matched by motifs for deoxysugars and halogenated aromatics upstream of a related BGC constitutes a candidate for the target metabolite's biosynthesis.

## Related tools

- **antiSMASH** (Detects putative BGC regions in the genome assembly and provides region identifiers that are then tokenised) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **iPRESTO** (Tokenises BGC genes by (sub)Pfam detection and queries them against annotated PRESTO-TOP sub-cluster motifs to flag candidate regions) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **build_subpfam** (Builds per-clade subPfam pHMM models from aligned protein sequences used for finer-grained gene tokenisation) — satriaphd/build_subpfam

## Examples

```
antismash --genefinding-tool prodigal QZWF01000007.fasta; ipresto query --motifs presto_top_1000.tsv --thresholds 0.3 1.1 0.15 --output akashin_bgc_subcluster_motifs.tsv
```

## Evaluation signals

- The report must show the expected region (e.g. QZWF01000007.1.region003, StrepF001_25985–StrepF001_26130) matched by motifs 194, 607, and 680, reproducing the candidate akashin BGC reported in the paper
- All matches must pass PRESTO-TOP thresholds: ≥0.3 gene-to-motif probability per gene, ≥1.1 summed gene-to-motif probability, and ≥0.15 overlap score
- Tokenisation must remove non-biosynthetic Pfams before motif querying; sub-clusters of length one should not be counted as they are almost definitely noise
- Sub-cluster predictions for validated reference BGCs (e.g. rifamycin BGC0000373) should reproduce expected motif hits with overlap ≥0.6 considered sufficient

## Limitations

- No version numbers stated for antiSMASH or iPRESTO in the supplementary documentation, so exact reproducibility of tool versions must be pinned separately
- Sub-clusters of length one are treated as noise and excluded, so single-gene substructural features may be missed
- Hybrid BGCs are counted for all classes they contain, which can inflate class-level match counts

## Evidence

- [supplementary] Tokenise each gene in the predicted BGCs with iPRESTO by detecting (sub)Pfam domains via HMMER3 hmmscan, removing non-biosynthetic Pfams.: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [supplementary] Sub-clusters of length one are almost certainly noise and must be discarded during motif matching.: "not counting sub-clusters of length one as these are almost definitely noise (see Methods)"
- [intro] iPRESTO's core purpose is unsupervised sub-cluster prediction via statistical co-occurrence and topic-modelling approaches.: "To predict sub-clusters, iPRESTO uses a statistical approach (PRESTO-STAT) as well as a topic modelling algorithm (PRESTO-TOP)."
- [intro] The akashin demonstration case shows how annotated motifs flag the candidate BGC region.: "showed direct usefulness of our method by correctly predicting the BGCs for xenorhabdin and salbostatin biosynthesis (which have been published but were missing from BGC databases) and identifying a"
- [readme] subPfam models are per-clade pHMMs derived from phylogenetic splitting of Pfam alignments.: [evidence span withheld — anchor status: external_doc]
