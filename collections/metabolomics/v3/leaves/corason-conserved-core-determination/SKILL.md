---
name: corason-conserved-core-determination
description: Use when a set of related BGCs (e.g., tauD-expansion clusters) needs
  a phylogeny-resolved organization beyond BiG-SCAPE GCF clustering, especially when
  fragmented assemblies or merged adjacent clusters split them into different GCFs.
when_to_use_negative:
- When the BGCs are so divergent that the conserved core collapses to only the query
  gene — the multi-locus alignment and phylogeny become uninformative.
- When the goal is only GCF-level similarity grouping — BiG-SCAPE network clustering
  with cutoffs (e.g., 0.3) is the appropriate, faster tool.
- When input gene clusters have not been processed by antiSMASH or indexed as RAST/GenBank
  files for CORASON.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: CORASON
  role: Runs the all-vs-all best bidirectional hit search that defines the conserved
    core and reconstructs the multi-locus phylogeny
  repo: https://github.com/nselem/corason
- name: BiG-SCAPE
  role: Upstream pipeline supplying antiSMASH GenBank BGC inputs and GCF context that
    CORASON phylogenies refine
  repo: https://git.wur.nl/medema-group/BiG-SCAPE
- name: hmmalign
  role: Aligns each predicted domain type of the conserved core using its corresponding
    HMM model to build the multi-locus alignment
- name: antiSMASH
  role: Produces the GenBank BGC predictions used directly as CORASON input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:22f222b224961e340f30894e996d751a52f2af110aa3ab10b1c2a9a1cbc84206
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/nselem/corason
  tools:
  - BiG-SCAPE
  - CORASON
  - antiSMASH
  - hmmalign
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# corason-conserved-core-determination

## Summary

Determines the conserved genomic core shared across a set of biosynthetic gene clusters using CORASON's all-vs-all best bidirectional hit search, then aligns the core's domain sequences (hmmalign) to build a multi-locus FastTree phylogeny that orders the BGCs evolutionarily. Captures cluster relationships that GCF-network clustering (e.g., BiG-SCAPE cutoffs) can miss.

## When to use

Use when a set of related BGCs (e.g., tauD-expansion clusters) needs a phylogeny-resolved organization beyond BiG-SCAPE GCF clustering, especially when fragmented assemblies or merged adjacent clusters split them into different GCFs.

## When NOT to use

- When the BGCs are so divergent that the conserved core collapses to only the query gene — the multi-locus alignment and phylogeny become uninformative.
- When the goal is only GCF-level similarity grouping — BiG-SCAPE network clustering with cutoffs (e.g., 0.3) is the appropriate, faster tool.
- When input gene clusters have not been processed by antiSMASH or indexed as RAST/GenBank files for CORASON.

## Inputs

- antiSMASH-processed GenBank files of the target BGC set (e.g., tauD-expansion BGCs)
- A query gene (e.g., tauD) as FASTA for CORASON
- RAST or GenBank genomic database of search-space genomes (e.g., actinobacterial genome set at Zenodo DOI 10.5281/zenodo.1532752)
- RAST_IDs tab-separated file (RAST mode) or gbk-indexed GenBank folder (gbk mode)

## Outputs

- query.svg — SVG graph with clusters sorted according to the multi-locus phylogeny of the common core
- query_Report — functional cluster genomic core report
- *.tre — phylogenetic tree (FastTree) of the genomic cluster core
- Multi-locus conserved-core alignment of domain sequences

## How to apply

Assemble the antiSMASH-processed GenBank BGCs and designate a single query gene (e.g., tauD for the detoxin/rimosamide supercluster clan); point CORASON at the actinobacterial genomic database deposited at Zenodo DOI 10.5281/zenodo.1532752 as the homolog search space. CORASON defines the conserved core by an all-vs-all best bidirectional hit search: a homologous gene is part of the core only if it is shared among all cluster variations in the internal database. The sequences of each predicted domain type in the core are then aligned with their corresponding hmmalign models to build a multi-locus alignment. If the alignment comes back empty (core collapsed to the query gene alone for very divergent BGCs), reduce the hmmer score cutoff or narrow the taxonomic diversity of the genomic database rather than abandoning the run. Finally, infer the multi-locus phylogeny with FastTree and inspect the tree for the expected clades; without the alignment step, output BGCs are drawn but not phylogenetically sorted.

## Related tools

- **CORASON** (Runs the all-vs-all best bidirectional hit search that defines the conserved core and reconstructs the multi-locus phylogeny) — https://github.com/nselem/corason
- **BiG-SCAPE** (Upstream pipeline supplying antiSMASH GenBank BGC inputs and GCF context that CORASON phylogenies refine) — https://git.wur.nl/medema-group/BiG-SCAPE
- **hmmalign** (Aligns each predicted domain type of the conserved core using its corresponding HMM model to build the multi-locus alignment)
- **antiSMASH** (Produces the GenBank BGC predictions used directly as CORASON input)

## Examples

```
docker run --rm -v $(pwd):/home/output nselem/corason:latest SSHcorason.pl yourquery.query yourRAST.Ids yourspecial_org
```

## Evaluation signals

- The FastTree multi-locus phylogeny resolves the three expected clades (P450/enoyl, supercluster, Amycolatopsis/P450) for the tauD-expansion detoxin/rimosamide clan.
- Every gene in the reported conserved core is a best bidirectional hit shared across all cluster variations in the internal database.
- The alignment is non-empty; an empty alignment signals the score cutoff or taxonomic scope must be reduced before results are interpretable.
- Output SVG shows BGCs sorted according to the phylogeny — if BGCs are drawn but not sorted, the alignment step was skipped or failed.
- The tree recovers tauD-expansion BGCs that were omitted from BiG-SCAPE GCFs due to fragmented assemblies or merged adjacent clusters.

## Limitations

- If the BGCs are not closely related, the core may be reduced to only the query gene, and without an alignment CORASON output BGCs are drawn but not sorted.
- If the alignment itself is empty, it is recommended to reduce the score cutoff or the scope of the taxonomic diversity on the genomic database (manual parameter adjustment).
- CORASON/EvoMining analyses in the study used a reduced set of 1,668 unpublished draft Actinobacterial genomes; results depend on the genome database supplied.
- Validation in the source study was qualitative (clade resolution and recovery of GCF-missed BGCs); no numeric threshold was reported.

## Evidence

- [readme] README: CORASON identifies common genomic core: [evidence span withheld — anchor status: external_doc]
- [readme] README: phylogenetically sorted SVG output: [evidence span withheld — anchor status: external_doc]
