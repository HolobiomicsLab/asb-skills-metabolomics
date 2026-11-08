---
name: genome-mining-bgc-detection
description: Use when starting from a raw genome assembly (e.g. a WGS contig such
  as QZWF01000007) and the goal is to enumerate putative BGC regions and obtain Pfam/subPfam
  tokenised genes ready for sub-cluster motif querying.
when_to_use_negative:
- When the goal is de novo sub-cluster discovery on a training corpus rather than
  flagging BGCs in a single query genome — use iPRESTO's PRESTO-STAT/PRESTO-TOP training
  workflow instead
- When the metabolite structure is already known and a matching MIBiG reference BGC
  exists — direct similarity-based BGC annotation (e.g. antiSMASH ClusterBlast/SubClusterBlast)
  suffices
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3174
tools:
- name: antiSMASH
  role: Detects putative BGC regions in the genome assembly and produces region identifiers
    that are then tokenised
- name: iPRESTO
  role: Tokenises BGC genes by (sub)Pfam domains (removing non-biosynthetic Pfams)
    and queries tokenised BGCs to the PRESTO-TOP sub-cluster motif set
  repo: https://git.wageningenur.nl/bioinformatics/iPRESTO/
- name: build_subpfam
  role: Builds per-clade subPfam pHMM models by cutting phylogenetic trees of Pfam
    alignments, providing finer-grained tokens for BGC tokenisation
  repo: https://github.com/satriaphd/build_subpfam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1010462
    title: 'iPRESTO: Automated discovery of biosynthetic sub-clusters linked to specific
      natural product substructures'
  source_content_hash: sha256:48bf0250de23bdf659adc5e17a56a5d1d5af31428490709e29907e03ed4e9756
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - antiSMASH
  - build_subpfam
  - iPRESTO
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# genome-mining-bgc-detection

## Summary

Detects putative biosynthetic gene clusters (BGCs) in a whole-genome shotgun assembly with antiSMASH and tokenises each gene by its (sub)Pfam domains via HMMER3 hmmscan, so that sub-cluster motifs (e.g. PRESTO-TOP motifs) can be queried to flag candidate BGCs for orphan metabolites such as the akashin BGC QZWF01000007.1.region003 in Streptomyces sp. F001.

## When to use

Use when starting from a raw genome assembly (e.g. a WGS contig such as QZWF01000007) and the goal is to enumerate putative BGC regions and obtain Pfam/subPfam tokenised genes ready for sub-cluster motif querying.

## When NOT to use

- When the goal is de novo sub-cluster discovery on a training corpus rather than flagging BGCs in a single query genome — use iPRESTO's PRESTO-STAT/PRESTO-TOP training workflow instead
- When the metabolite structure is already known and a matching MIBiG reference BGC exists — direct similarity-based BGC annotation (e.g. antiSMASH ClusterBlast/SubClusterBlast) suffices

## Inputs

- Whole-genome shotgun nucleotide assembly (e.g. contig QZWF01000007 from Streptomyces sp. F001) from the public nucleotide archive
- Annotated PRESTO-TOP sub-cluster motif set (S1 File; 1,000 motifs with annotations such as motif 194 = methylaminosugar)
- subPfam pHMM library (built for the 112 most frequent Pfam domains in antiSMASH-DB via build_subpfam)

## Outputs

- antiSMASH BGC region identifiers and boundaries (e.g. QZWF01000007.1.region003 spanning StrepF001_25985–StrepF001_26130)
- Tokenised BGCs: per-gene (sub)Pfam domain lists with non-biosynthetic Pfams removed
- Per-region motif-match table (e.g. akashin_bgc_subcluster_motifs.tsv) listing matched sub-cluster motifs and their annotations

## How to apply

Download the whole-genome shotgun assembly containing the contig of interest from the public nucleotide archive and run antiSMASH (version 6 in the reproduced task) on it to detect putative BGCs, retaining region identifiers such as QZWF01000007.1.region003. Tokenise each gene in every predicted BGC with iPRESTO by detecting (sub)Pfam domains via HMMER3 hmmscan, removing non-biosynthetic Pfams so only biosynthetic tokens remain. Optionally build per-clade subPfam pHMMs with build_subpfam, which cuts a mid-rooted, laderized, ultrametricized phylogenetic tree of each Pfam multiple sequence alignment at a specified length point. Query the tokenised BGCs against the annotated 1,000 PRESTO-TOP sub-cluster motif set applying the thresholds of 0.3 gene-to-motif probability per gene, 1.1 summed gene-to-motif probability, and 0.15 overlap score, then record which regions match annotated motifs (e.g. 194 methylaminosugar, 607 halogenated aromatic ring, 680 (amino)deoxysugar) and write the per-region motif-match table. Candidate BGCs are judged by co-occurrence of motifs encoding the expected substructures — for akashin, the region QZWF01000007.1.region003 (StrepF001_25985–StrepF001_26130), directly upstream of the diazaquinomycin BGC, is flagged by motifs 194, 607 and 680.

## Related tools

- **antiSMASH** (Detects putative BGC regions in the genome assembly and produces region identifiers that are then tokenised)
- **iPRESTO** (Tokenises BGC genes by (sub)Pfam domains (removing non-biosynthetic Pfams) and queries tokenised BGCs to the PRESTO-TOP sub-cluster motif set) — https://git.wageningenur.nl/bioinformatics/iPRESTO/
- **build_subpfam** (Builds per-clade subPfam pHMM models by cutting phylogenetic trees of Pfam alignments, providing finer-grained tokens for BGC tokenisation) — https://github.com/satriaphd/build_subpfam

## Examples

```
antismash --genefinding-glimmer QZWF01000007.fasta -o antismash_out && python -m ipresto tokenise --input antismash_out && python -m ipresto query --subclusters presto_top_motifs.tsv --output akashin_bgc_subcluster_motifs.tsv
```

## Evaluation signals

- antiSMASH run reports the expected region, e.g. QZWF01000007.1.region003 spanning StrepF001_25985–StrepF001_26130
- Motif query matches motifs 194, 607 and 680 in that region, reproducing the candidate akashin BGC
- PRESTO-TOP query thresholds respected: 0.3 gene-to-motif probability per gene, 1.1 summed gene-to-motif probability, 0.15 overlap score
- Tokenisation output contains no non-biosynthetic Pfam tokens (they are removed during tokenisation)
- Per-region motif-match table (e.g. akashin_bgc_subcluster_motifs.tsv) is written with motif annotations consistent with the S1 File

## Limitations

- No version numbers stated for antiSMASH or iPRESTO in the source material, so exact reproducibility of domain detection may vary
- Sub-clusters of length one are almost definitely noise and should not be counted; matches of length 1 are ignored and hybrid class BGCs count for all classes they contain
- iPRESTO only predicted a substructure for 16% of antiSMASH-DB BGCs, so a negative motif match does not exclude a candidate BGC
- build_subpfam README is under construction ('Usage: under construction'), so parameterisation must be inferred from the paper's methods

## Evidence

- [supplementary] BGC detection and tokenisation with antiSMASH + iPRESTO: "The BGCs predicted by antiSMASH are tokenised by detecting (sub)Pfams in each gene, where non-biosynthetic Pfams are removed"
- [results] Candidate BGC identification validated by motif matches: "correctly identify the BGCs for xenorhabdin and salbostatin biosynthesis (which had not yet been annotated in BGC databases), as well as propose a candidate BGC for akashin biosynthesis"
- [supplementary] LDA bag-of-words tokenisation assumption: "or by LDA, which assumes a bag of words model where original gene order is not considered"
- [readme] subPfam construction from phylogenetic clades: [evidence span withheld — anchor status: external_doc]
- [intro] iPRESTO availability: "iPRESTO is available as a command-line tool at https://git.wageningenur.nl/bioinformatics/iPRESTO/."
