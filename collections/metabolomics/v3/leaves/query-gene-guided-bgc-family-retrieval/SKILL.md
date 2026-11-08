---
name: query-gene-guided-bgc-family-retrieval
description: Use when a single anchor gene defines a BGC family of interest and GCF
  clustering (BiG-SCAPE distance cutoffs) has omitted fragmented or merged assemblies
  from the family; needs evolutionary ordering of related clusters.
when_to_use_negative:
- When the goal is network-based GCF assignment across many BGC classes rather than
  one query-gene family — BiG-SCAPE cutoff clustering (e.g. 0.3) is the appropriate
  first pass; CORASON is the follow-up for evolutionary resolution within/ across
  a family
- When BGCs are so divergent that no conserved core beyond the query gene can be found
  — the alignment will be empty or query-only and the output BGCs will be drawn but
  not phylogenetically sorted
- When no curated genomic database or assembled genomes exist for the taxonomic space
  of interest
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0080
tools:
- name: CORASON
  role: Runs the query-gene all-vs-all best bidirectional hit search, determines the
    conserved core, and outputs the SVG cluster graph sorted by the multi-locus phylogeny
  repo: https://github.com/nselem/corason
- name: hmmalign
  role: Aligns sequences of every predicted domain type using each corresponding HMM
    model to build the multi-locus core alignment
- name: antiSMASH
  role: Produces the GenBank BGC prediction files used as CORASON input
- name: BiG-SCAPE
  role: Upstream GCF network clustering whose omitted family members the query-gene
    CORASON phylogeny recovers; also provides CORASON-like trees in its HTML output
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:47a6a6d94d293fe2e7ab01fe9acf1063750b57bf9b54a5e7dcf6d0de20ad4962
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/nselem/corason
  tools:
  - BiG-SCAPE
  - CORASON
  - antiSMASH
  - hmmalign
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Query-gene-guided BGC family retrieval and multi-locus phylogeny with CORASON

## Summary

Designates one biosynthetic gene (e.g. tauD) as a query, retrieves homologous gene clusters from a genomic database via all-vs-all best bidirectional hits, and sorts them by a multi-locus phylogeny of the conserved core — recovering BGC family members (such as detoxin/rimosamide-related clusters) that BiG-SCAPE GCF clustering at standard cutoffs can miss.

## When to use

Use when a single anchor gene defines a BGC family of interest and GCF clustering (BiG-SCAPE distance cutoffs) has omitted fragmented or merged assemblies from the family; needs evolutionary ordering of related clusters.

## When NOT to use

- When the goal is network-based GCF assignment across many BGC classes rather than one query-gene family — BiG-SCAPE cutoff clustering (e.g. 0.3) is the appropriate first pass; CORASON is the follow-up for evolutionary resolution within/ across a family
- When BGCs are so divergent that no conserved core beyond the query gene can be found — the alignment will be empty or query-only and the output BGCs will be drawn but not phylogenetically sorted
- When no curated genomic database or assembled genomes exist for the taxonomic space of interest

## Inputs

- Query gene amino-acid FASTA (file.query), e.g. tauD
- Genomic database of antiSMASH-processed GenBank BGC files (gbk mode, indexed with gbkIndex.pl) or a RAST genome database with a RAST_IDs tab-separated file
- Actinobacterial genomic database from Zenodo DOI 10.5281/zenodo.1532752 used as the homolog search space
- Optional special organism identifier (-s) to prioritize in the output

## Outputs

- query.svg — SVG graph of homologous clusters sorted according to the conserved-core multi-locus phylogeny
- query_Report — functional cluster genomic core report
- *.tre — phylogenetic tree of the genomic cluster core
- Conserved-core gene sets defined by all-vs-all best bidirectional hits across cluster variations

## How to apply

Assemble the query-expansion BGC set as GenBank files and designate the anchor gene (e.g. tauD) as the query; index GenBank input with gbkIndex.pl or provide a RAST genome database plus RAST_IDs file. Point CORASON at a broad genomic search space (here the actinobacterial database deposited at Zenodo DOI 10.5281/zenodo.1532752, or the reduced 1,668-genome Actinobacterial set) and run the all-vs-all best bidirectional hit search to define the conserved core of homologous genes shared across candidate BGCs. Align each conserved-core domain type with its corresponding hmmalign model to build the multi-locus alignment, then infer the family phylogeny with FastTree from the curated core alignment (curation via Gblocks: minimum block length 5, max 10 contiguous non-conserved positions, gaps in <50% of sequences). Inspect the tree for biologically meaningful clades and for BGCs absent from BiG-SCAPE GCFs. If the alignment is empty, reduce the score cutoff or the taxonomic scope of the genomic database, since for divergent BGCs the conserved core may collapse to only the query gene.

## Related tools

- **CORASON** (Runs the query-gene all-vs-all best bidirectional hit search, determines the conserved core, and outputs the SVG cluster graph sorted by the multi-locus phylogeny) — https://github.com/nselem/corason
- **hmmalign** (Aligns sequences of every predicted domain type using each corresponding HMM model to build the multi-locus core alignment)
- **antiSMASH** (Produces the GenBank BGC prediction files used as CORASON input)
- **BiG-SCAPE** (Upstream GCF network clustering whose omitted family members the query-gene CORASON phylogeny recovers; also provides CORASON-like trees in its HTML output) — https://git.wageningenur.nl/medema-group/BiG-SCAPE

## Examples

```
docker run --rm -v $(pwd):/home/output nselem/corason:latest SSHcorason.pl tauD.query RAST.Ids yourspecial_org
```

## Evaluation signals

- The FastTree multi-locus phylogeny resolves the three expected clades for the detoxin/rimosamide supercluster clan: P450/enoyl, supercluster, and Amycolatopsis/P450 (qualitative criterion; no numeric threshold reported)
- The tree recovers tauD-expansion BGCs that BiG-SCAPE GCF clustering omitted (fragmented assemblies or merged adjacent clusters pushed into different GCFs)
- A conserved core of homologous genes is identified by best bidirectional hits and shared across cluster variations, not reduced to only the query gene
- Output directory contains query.svg, query_Report, and *.tre files with BGCs phylogenetically sorted (not drawn unsorted)
- CORASON/EvoMining analyses are reproducible from the deposited Zenodo DOI 10.5281/zenodo.1532752 data

## Limitations

- If BGCs are not closely related, the conserved core may be reduced to only the query gene, weakening the phylogeny
- An empty alignment failure mode requires manually reducing the score cutoff or the scope of taxonomic diversity on the genomic database; without an alignment, CORASON output BGCs are drawn but not sorted
- The validation criterion is qualitative — the study reports no numeric threshold for phylogeny resolution
- CORASON analyses in this study relied on unpublished draft genomes (a reduced set of 1,668 Actinobacterial genomes), limiting exact reproducibility outside the Zenodo deposit

## Evidence

- [readme] CORASON is a visual tool that identifies gene clusters that share a common genomic core and reconstructs multi-locus phylogenies of these gene clusters: [evidence span withheld — anchor status: external_doc]
- [readme] Input:** query gene and RAST genome database.: [evidence span withheld — anchor status: external_doc]
