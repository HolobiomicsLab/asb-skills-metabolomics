---
name: gcf-versus-phylogeny-discrepancy-analysis
description: Use when BiG-SCAPE GCF clustering may have missed related BGCs — e.g.,
  fragmented assemblies or merged adjacent clusters split a known family — and an
  evolutionary, query-gene-anchored view is needed to recover them.
when_to_use_negative:
- When BGCs are too divergent from the query — the CORASON conserved core may collapse
  to only the query gene, leaving no multi-locus alignment to sort by
- When the goal is genome-wide GCF calling rather than resolving relationships within
  one targeted family — BiG-SCAPE network clustering at cutoff 0.3 is the appropriate
  scalable tool
- When the CORASON alignment comes back empty (score cutoff too strict or taxonomic
  scope too broad) without manual parameter reduction — output BGCs would be drawn
  but not sorted
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: CORASON
  role: Computes the all-vs-all best bidirectional hit conserved core and the multi-locus
    phylogeny anchored on the query gene (tauD)
  repo: https://github.com/nselem/corason
- name: antiSMASH v4
  role: Produces the GenBank BGC predictions used directly as CORASON input (--minimal
    parameters)
- name: hmmalign
  role: Aligns each predicted domain type against its corresponding HMM model to build
    the multi-locus core alignment
- name: BiG-SCAPE
  role: Provides the GCF clustering baseline (network cutoff 0.3) against which phylogeny
    members are compared to find omissions
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:2591da7916d40b376acc92b6682b3ebd80ef1a5fc901d95c57ef31c3e5dd1a44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/nselem/corason
  tools:
  - BiG-SCAPE
  - CORASON
  - antiSMASH v4
  - hmmalign
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gcf-versus-phylogeny-discrepancy-analysis

## Summary

Builds a CORASON multi-locus phylogeny around a query gene to identify BGCs that belong to a family (e.g., the detoxin/rimosamide supercluster clan) but were pushed into different BiG-SCAPE GCFs by fragmented assemblies or merged adjacent clusters under the 0.3 distance cutoff.

## When to use

Use when BiG-SCAPE GCF clustering may have missed related BGCs — e.g., fragmented assemblies or merged adjacent clusters split a known family — and an evolutionary, query-gene-anchored view is needed to recover them.

## When NOT to use

- When BGCs are too divergent from the query — the CORASON conserved core may collapse to only the query gene, leaving no multi-locus alignment to sort by
- When the goal is genome-wide GCF calling rather than resolving relationships within one targeted family — BiG-SCAPE network clustering at cutoff 0.3 is the appropriate scalable tool
- When the CORASON alignment comes back empty (score cutoff too strict or taxonomic scope too broad) without manual parameter reduction — output BGCs would be drawn but not sorted

## Inputs

- antiSMASH-predicted BGC GenBank files (e.g., 1,175 tauD-expansion BGCs)
- Query-gene amino-acid FASTA file (e.g., file.query containing tauD)
- Actinobacterial genomic reference database (Zenodo DOI 10.5281/zenodo.1532752)
- Pfam database version 31 for domain detection with HMMER 3.1b2
- Optional BiG-SCAPE GCF assignments (cutoff 0.3) to diff against the phylogeny

## Outputs

- FastTree multi-locus phylogeny of the conserved core (*.tre)
- CORASON SVG graph with BGCs sorted according to the multi-locus phylogeny (query.svg)
- Conserved-core functional report (query_Report)
- List of BGCs present in the phylogeny but absent from the BiG-SCAPE GCF

## How to apply

Assemble the BGC set of interest as GenBank files (here, the 1,175 tauD-expansion BGCs predicted with antiSMASH v4 --minimal) and designate the anchor gene (tauD) as the CORASON query. Point CORASON at the Actinobacterial genomic database deposited at Zenodo DOI 10.5281/zenodo.1532752 for all-vs-all best bidirectional hit retrieval of the conserved core — the core is defined as homologous genes shared among the cluster variations. Align each predicted domain type against its corresponding HMM model with hmmalign (HMMER 3.1b2, Pfam version 31) to build the multi-locus alignment, then infer the phylogeny with FastTree. Inspect the tree for the expected resolution — for the detoxin/rimosamide clan, the P450/enoyl, supercluster, and Amycolatopsis/P450 clades — and list tree members absent from the BiG-SCAPE GCF (network filtered at cutoff 0.3) as clustering omissions. Acceptance is qualitative: the FastTree tree must resolve the expected clades and recover tauD-expansion BGCs omitted by GCF clustering; the study reports no numeric threshold.

## Related tools

- **CORASON** (Computes the all-vs-all best bidirectional hit conserved core and the multi-locus phylogeny anchored on the query gene (tauD)) — https://github.com/nselem/corason
- **antiSMASH v4** (Produces the GenBank BGC predictions used directly as CORASON input (--minimal parameters))
- **hmmalign** (Aligns each predicted domain type against its corresponding HMM model to build the multi-locus core alignment)
- **BiG-SCAPE** (Provides the GCF clustering baseline (network cutoff 0.3) against which phylogeny members are compared to find omissions) — https://git.wageningenur.nl/medema-group/BiG-SCAPE

## Examples

```
docker run --rm -v $(pwd):/home/output nselem/corason:latest SSHcorason.pl tauD.query RAST.Ids yourspecial_org
```

## Evaluation signals

- The FastTree multi-locus phylogeny resolves the three expected clades (P450/enoyl, supercluster, Amycolatopsis/P450) for the detoxin/rimosamide clan
- The tree recovers tauD-expansion BGCs that were assigned to different GCFs by BiG-SCAPE at the 0.3 cutoff due to fragmented assemblies or merged adjacent clusters
- A non-empty multi-locus alignment exists — the conserved core contains more than just the query gene
- Output folder contains query.svg, query_Report, and *.tre files with BGCs sorted (not merely drawn) according to the tree

## Limitations

- If the BGCs are not closely related, the conserved core may be reduced to only the query gene
- If the alignment itself is empty, it is recommended to reduce the score cutoff or the scope of the taxonomic diversity on the genomic database
- Without an alignment, CORASON output BGCs are drawn but not sorted
- The CORASON/EvoMining analyses used a reduced set of 1,668 unpublished draft Actinobacterial genomes, so coverage depends on the supplied database

## Evidence

- [readme] CORASON output description from README: [evidence span withheld — anchor status: external_doc]
