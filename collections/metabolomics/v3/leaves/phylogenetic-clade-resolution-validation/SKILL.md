---
name: phylogenetic-clade-resolution-validation
description: Use when GCF clustering (e.g., BiG-SCAPE) may have omitted or misassigned
  related BGCs due to fragmented assemblies or merged adjacent clusters, and you need
  a query-gene-rooted phylogeny to recover them.
when_to_use_negative:
- BGCs are so divergent that the conserved core collapses to only the query gene —
  without a multi-locus alignment the output BGCs are drawn but not sorted, so clade
  validation is meaningless
- The goal is global GCF/GCC assignment by pairwise domain-content distance rather
  than evolutionary resolution within one family — BiG-SCAPE (Jaccard/domain-sequence/adjacency
  distances, Affinity Propagation) is the appropriate tool
- The alignment comes out empty and reducing the hmmer score cutoff or taxonomic scope
  of the genomic database is not acceptable for the analysis
edam_operation: http://edamontology.org/operation_0554
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0084
tools:
- name: CORASON
  role: Computes the all-vs-all best bidirectional hit conserved core and sorts clusters
    by the multi-locus phylogeny of the query gene
  repo: https://github.com/nselem/corason
- name: hmmalign
  role: Model-based alignment of each conserved-core domain type to build the multi-locus
    alignment
- name: FastTree
  role: Infers the high-resolution multi-locus phylogeny from the curated core alignment
- name: antiSMASH
  role: Provides the BGC GenBank inputs and 'Core Biosynthetic' gene annotations used
    in core definition
  repo: https://antismash.secondarymetabolites.org
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:4fcabcf38c828faec010725872302500aa9e49f1e65d4f1e0c208af3f7235cd4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/nselem/corason
  tools:
  - CORASON
  - FastTree
  - antiSMASH
  - hmmalign
  edam_operation: http://edamontology.org/operation_0554
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# phylogenetic-clade-resolution-validation

## Summary

Builds a high-resolution multi-locus phylogeny of BGCs sharing a query gene with CORASON (all-vs-all best bidirectional hits → hmmalign domain alignment → FastTree) and validates the result by qualitative clade resolution rather than numeric cutoffs.

## When to use

Use when GCF clustering (e.g., BiG-SCAPE) may have omitted or misassigned related BGCs due to fragmented assemblies or merged adjacent clusters, and you need a query-gene-rooted phylogeny to recover them.

## When NOT to use

- BGCs are so divergent that the conserved core collapses to only the query gene — without a multi-locus alignment the output BGCs are drawn but not sorted, so clade validation is meaningless
- The goal is global GCF/GCC assignment by pairwise domain-content distance rather than evolutionary resolution within one family — BiG-SCAPE (Jaccard/domain-sequence/adjacency distances, Affinity Propagation) is the appropriate tool
- The alignment comes out empty and reducing the hmmer score cutoff or taxonomic scope of the genomic database is not acceptable for the analysis

## Inputs

- antiSMASH-processed GenBank files of the BGC set (e.g., tauD-expansion BGCs)
- Query gene aminoacid FASTA file (file.query)
- Genomic database (e.g., actinobacterial GenBank/RAST genomes from Zenodo DOI 10.5281/zenodo.1532752)
- Pfam database (version 31 used with HMMER 3.1b2) for domain prediction

## Outputs

- Multi-locus phylogenetic tree (.tre) of the conserved core
- SVG graph with clusters sorted according to the core phylogeny
- Conserved-core functional report (query_Report)
- Curated multi-locus domain alignment from hmmalign

## How to apply

Assemble the BGC set (antiSMASH-processed GenBank files) and designate the anchor gene (e.g., tauD in the detoxin/rimosamide supercluster clan) as the query. Run CORASON's all-vs-all best bidirectional hit search against the genomic database (e.g., the 1,668-genome Actinobacterial set deposited at Zenodo DOI 10.5281/zenodo.1532752) to define the conserved core: homologous genes shared across the cluster variations are retained only if the longest common segment (LCS) is three genes long or contains a gene marked 'Core Biosynthetic' by antiSMASH. Align each predicted domain type with its corresponding model using hmmalign (chosen because runtimes stay short as input BGC count grows), then infer the multi-locus phylogeny with FastTree. Validate qualitatively: the tree must resolve the expected clades — for the tauD-expansion set, the P450/enoyl, supercluster, and Amycolatopsis/P450 clades — and recover BGCs that BiG-SCAPE GCF clustering (cutoff 0.3) omitted; no numeric threshold applies.

## Related tools

- **CORASON** (Computes the all-vs-all best bidirectional hit conserved core and sorts clusters by the multi-locus phylogeny of the query gene) — https://github.com/nselem/corason
- **hmmalign** (Model-based alignment of each conserved-core domain type to build the multi-locus alignment)
- **FastTree** (Infers the high-resolution multi-locus phylogeny from the curated core alignment)
- **antiSMASH** (Provides the BGC GenBank inputs and 'Core Biosynthetic' gene annotations used in core definition) — https://antismash.secondarymetabolites.org

## Examples

```
docker run --rm -v $(pwd):/home/output nselem/corason:latest SSHcorason.pl tauD.query yourRAST.Ids yourspecial_org
```

## Evaluation signals

- The FastTree multi-locus phylogeny resolves the expected clades (P450/enoyl, supercluster, Amycolatopsis/P450 for the tauD-expansion set)
- The tree recovers tauD-expansion BGCs that BiG-SCAPE GCF clustering had omitted due to fragmented assemblies or merged adjacent clusters
- Output files exist in the query output folder: query.svg, query_Report, and *.tre
- The conserved core contains more than the query gene alone; an empty or single-gene core indicates the analysis must be redone with a reduced score cutoff or narrower taxonomic diversity
- GCF calling from the hmmalign-based pipeline agrees with Muscle/MAFFT alternatives (cross-method sanity check reported in the article)

## Limitations

- For divergent BGCs the conserved core may be reduced to only the query gene, leaving BGCs drawn but not sorted
- If the alignment itself is empty, the score cutoff or the taxonomic diversity scope of the genomic database must be manually reduced
- Validation is qualitative — the study reports no numeric threshold for correct clade resolution
- CORASON analyses in the article used unpublished draft genomes (a reduced set of 1,668 Actinobacterial genomes), affecting exact reproducibility

## Evidence

- [readme] README description of CORASON output: [evidence span withheld — anchor status: external_doc]
