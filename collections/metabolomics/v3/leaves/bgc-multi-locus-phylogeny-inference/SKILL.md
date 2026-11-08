---
name: bgc-multi-locus-phylogeny-inference
description: Use when antiSMASH-processed GenBank BGCs plus a chosen query gene are
  available and network-based GCF clustering (BiG-SCAPE) is suspected to have split
  or omitted related clusters — e.
when_to_use_negative:
- 'When the BGCs under comparison are not closely related: CORASON''s conserved core
  can collapse to only the query gene, and an empty alignment forces manual reduction
  of the score cutoff or taxonomic scope'
- When a coarse grouping rather than evolutionary ordering suffices — BiG-SCAPE GCF/GCC
  network clustering (cutoff 0.75 default, clan cutoff 0.3) is far faster and may
  be adequate
- When inputs are raw contigs or RAST annotations incompatible with the CORASON GenBank
  workflow (gbk files must be indexed first via gbkIndex.pl)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: CORASON
  role: computes the conserved genomic core via all-vs-all best bidirectional hits
    and renders cluster SVGs sorted by the multi-locus phylogeny
  repo: https://github.com/nselem/corason
- name: hmmalign
  role: model-based alignment of each predicted domain type to build the multi-locus
    alignment fed to tree inference
- name: antiSMASH
  role: produces the GenBank BGC predictions used directly as CORASON/BiG-SCAPE input
    (v4 with --minimal in the study)
- name: BiG-SCAPE
  role: complementary network GCF clustering used as baseline; CORASON runs on GCF
    members to add phylogenetic resolution
  repo: https://git.wur.nl/medema-group/BiG-SCAPE
- name: Gblocks
  role: alignment curation (min block length 5, max 10 contiguous non-conserved positions,
    gaps in <50% of sequences) before tree inference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:ab86895f14a7b32c206388b771eaae2b7e44f07e85d7dd0593c7144896f4b11a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/nselem/corason
  tools:
  - BiG-SCAPE
  - CORASON
  - Gblocks
  - antiSMASH
  - hmmalign
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# bgc-multi-locus-phylogeny-inference

## Summary

Infers high-resolution multi-locus phylogenies of biosynthetic gene clusters by running CORASON on a designated query gene over antiSMASH-processed GenBank BGCs, using all-vs-all best bidirectional hits to define a conserved genomic core, aligning core domains per Pfam model with hmmalign, and building a FastTree phylogeny that resolves relationships BiG-SCAPE GCF clustering misses.

## When to use

Use when antiSMASH-processed GenBank BGCs plus a chosen query gene are available and network-based GCF clustering (BiG-SCAPE) is suspected to have split or omitted related clusters — e.g. fragmented assemblies or merged adjacent clusters — and an evolutionary (phylogenomic) ordering of the BGC family is needed.

## When NOT to use

- When the BGCs under comparison are not closely related: CORASON's conserved core can collapse to only the query gene, and an empty alignment forces manual reduction of the score cutoff or taxonomic scope
- When a coarse grouping rather than evolutionary ordering suffices — BiG-SCAPE GCF/GCC network clustering (cutoff 0.75 default, clan cutoff 0.3) is far faster and may be adequate
- When inputs are raw contigs or RAST annotations incompatible with the CORASON GenBank workflow (gbk files must be indexed first via gbkIndex.pl)

## Inputs

- antiSMASH-processed GenBank files of target BGCs (CORASON gbk mode; GenBank output of antiSMASH used directly)
- Query-gene specification (file.query amino-acid FASTA per the README)
- Reference genomic database in CORASON format (e.g. Zenodo DOI 10.5281/zenodo.1532752 Actinobacterial genomes; GENOMES dir plus RAST_IDs file in RAST mode)
- Pfam version 31 database for hmmer/HMMER 3.1b2 domain detection

## Outputs

- query.svg — clusters sorted according to the multi-locus phylogeny of the common core
- query_Report — functional cluster genomic core report
- *.tre — phylogenetic tree of the genomic cluster core
- Conserved-core gene set and multi-domain alignment underlying the tree

## How to apply

Assemble antiSMASH-processed GenBank files of the target BGC set (e.g. the 1,175 tauD-expansion BGCs) plus a genomic database such as the 1,668-Actinobacteria set at Zenodo DOI 10.5281/zenodo.1532752, designate the query gene (e.g. tauD, a P450/enoyl-style anchor), and run CORASON's all-vs-all best bidirectional hit search so a gene joins the conserved core only if it is shared across the cluster-variation internal database. Domain detection uses a user-supplied Pfam database (v31 with HMMER 3.1b2 in the reference study); overlapping domains are filtered by hmmer score. Align sequences of every predicted domain type against its own model with hmmalign (validated in the paper to give GCF calls agreeing with Muscle/MAFFT and shorter runtimes on large inputs), optionally curating with Gblocks (min block length 5, max 10 contiguous non-conserved positions, gaps in <50% of sequences), then infer the multi-locus tree with FastTree. Decide whether to run CORASON directly or via its Docker image (nselem/corason) for reproducibility; inspect the resulting SVG/tree for expected clade structure.

## Related tools

- **CORASON** (computes the conserved genomic core via all-vs-all best bidirectional hits and renders cluster SVGs sorted by the multi-locus phylogeny) — https://github.com/nselem/corason
- **hmmalign** (model-based alignment of each predicted domain type to build the multi-locus alignment fed to tree inference)
- **antiSMASH** (produces the GenBank BGC predictions used directly as CORASON/BiG-SCAPE input (v4 with --minimal in the study))
- **BiG-SCAPE** (complementary network GCF clustering used as baseline; CORASON runs on GCF members to add phylogenetic resolution) — https://git.wur.nl/medema-group/BiG-SCAPE
- **Gblocks** (alignment curation (min block length 5, max 10 contiguous non-conserved positions, gaps in <50% of sequences) before tree inference)

## Examples

```
docker run --rm -v $(pwd):/home/output nselem/corason:latest SSHcorason.pl yourquery.query yourRAST.Ids yourspecial_org
```

## Evaluation signals

- The FastTree multi-locus tree resolves the three expected clades (P450/enoyl, supercluster, Amycolatopsis/P450) for the tauD-family analysis
- The CORASON output contains query.svg, query_Report, and one or more *.tre files in /mydir/<query>
- Tree recovers tauD-expansion BGCs omitted by BiG-SCAPE GCF clustering (fragmented or merged-adjacent-cluster cases) — qualitative acceptance criterion in the study
- Conserved core is not degenerate: it contains more than the single query gene, confirming the BBH-derived core is shared across cluster variations
- A non-empty multi-domain alignment exists; empty alignment signals the score cutoff or taxonomic scope of the genomic database must be reduced

## Limitations

- Validation is qualitative — no numeric threshold for clade resolution or recovery of missed BGCs is reported in the study
- If BGCs are not closely related, the conserved core may shrink to only the query gene; without an alignment CORASON still draws BGCs but does not sort them
- Very large genomic databases hit CORASON's lengthy CPU-time profile noted as a general limitation of prior approaches; homolog retrieval quality depends on the deposited database scope (the study's CORASON analyses used a reduced set of 1,668 Actinobacterial genomes)

## Evidence

- [intro] CORASON employs a phylogenomic approach to elucidate evolutionary relationships between gene clusters by computing high-resolution multi-locus phylogenies of BGCs within and across GCFs: "CORASON employs a phylogenomic approach to elucidate evolutionary relationships between gene clusters by computing high-resolution multi-locus phylo"
- [methods] A set of homologous genes are considered part of the conserved-core if and only if they are shared among the cluster variations internal database: "A set of homologous genes are considered part of the conserved-core if and only if they are shared among the cluster variations internal database"
- [methods] The sequences of every predicted domain type are aligned using each corresponding model by hmmalign.: "The sequences of every predicted domain type are aligned using each corresponding model by hmmalign."
- [discussion] if the BGCs are not closely related, the core may be reduced to only the query gene: "if the BGCs are not closely related, the core may be reduced to only the query gene"
- [readme] CORASON is a visual tool that identifies gene clusters that share a common genomic core and reconstructs multi-locus phylogenies of these gene clusters: [evidence span withheld — anchor status: external_doc]
