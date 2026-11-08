---
name: genome-set-retrieval-from-ncbi
description: 'Use when building the raw-genome foundation for a pan-biosynthetic analysis:
  you need complete (or WGS draft) genomes for a taxon from NCBI, with outgroup genomes,
  before BGC prediction and GCF clustering.'
when_to_use_negative:
- Input is already antiSMASH GenBank output or a BiG-SCAPE-ready BGC set — no genome
  retrieval is needed
- The genomes of interest are unpublished draft assemblies not present in NCBI (as
  with the CORASON/EvoMining draft genomes in the paper)
- The goal is single-genome BGC annotation rather than a multi-genome absence/presence
  analysis
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SPAdes
  role: Assembles WGS draft genome reads retrieved via NCBI BioProject/SRA into usable
    assemblies when complete genomes are unavailable
- name: BiG-SCAPE
  role: 'Downstream consumer: the retrieved genome set is processed with antiSMASH
    and BiG-SCAPE; duplicate ORGANISM identifiers in the retrieved set cause genome-count
    collapse'
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: CORASON
  role: Downstream consumer of the retrieved (or separately sequenced draft) genomes
    in RAST or genbank mode
  repo: https://github.com/nselem/corason
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:7d294962c6311754294af98a9cd15691093b6e5487319ddcabafafa2819a5000
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - CORASON
  - SPAdes
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# genome-set-retrieval-from-ncbi

## Summary

Retrieves a complete, taxonomically coherent genome set from NCBI (e.g. 103 complete Streptomyces genomes plus outgroups such as CP001700 Catenulispora acidiphila and CP000850 Salinispora arenicola) to serve as the input basis for antiSMASH/BiG-SCAPE biosynthetic diversity analyses.

## When to use

Use when building the raw-genome foundation for a pan-biosynthetic analysis: you need complete (or WGS draft) genomes for a taxon from NCBI, with outgroup genomes, before BGC prediction and GCF clustering.

## When NOT to use

- Input is already antiSMASH GenBank output or a BiG-SCAPE-ready BGC set — no genome retrieval is needed
- The genomes of interest are unpublished draft assemblies not present in NCBI (as with the CORASON/EvoMining draft genomes in the paper)
- The goal is single-genome BGC annotation rather than a multi-genome absence/presence analysis

## Inputs

- NCBI taxon query (e.g. 'Streptomyces') filtered to complete genomes
- Explicit outgroup GenBank accessions (CP001700, CP000850)
- NCBI BioProject accession for WGS draft assemblies (e.g. PRJNA488366)
- SRA run accessions (e.g. SRX4638772-SRX4639021) for unassembled sequencing data

## Outputs

- Local set of genome records (FASTA/GenBank) with accession provenance for 103 Streptomyces + 2 outgroup genomes
- Accession-to-genome mapping table keyed by NCBI accession/ORGANISM identifier

## How to apply

Query NCBI for the target taxon, restricting to complete genomes (the article's example uses 103 complete Streptomyces genomes), and add explicit outgroup accession-based downloads (e.g. CP001700, CP000850). For WGS material, query NCBI for 'Whole genome shotgun sequencing project' entries as the article did for 2,831 Actinobacterial genomes, and assemble drafts with SPAdes using default options (e.g. BioProject PRJNA488366, SRA runs SRX4638772-SRX4639021). Track accession numbers for every genome so the downstream absence/presence matrix can be keyed per genome. Beware duplicate ORGANISM identifiers across assembly projects: the paper found BiG-SCAPE's interactive report collapsed 103 Streptomyces genomes to 96 because scaffolds sharing an ORGANISM identifier are grouped, so normalize organism naming at retrieval time.

## Related tools

- **SPAdes** (Assembles WGS draft genome reads retrieved via NCBI BioProject/SRA into usable assemblies when complete genomes are unavailable)
- **BiG-SCAPE** (Downstream consumer: the retrieved genome set is processed with antiSMASH and BiG-SCAPE; duplicate ORGANISM identifiers in the retrieved set cause genome-count collapse) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **CORASON** (Downstream consumer of the retrieved (or separately sequenced draft) genomes in RAST or genbank mode) — https://github.com/nselem/corason

## Evaluation signals

- Genome count matches the study design: 103 complete Streptomyces genomes plus 2 outgroups (105 analyzed genomes total)
- Every retrieved genome carries a resolvable NCBI accession (e.g. CP001700, CP000850) recorded in a provenance table
- WGS-derived drafts trace back to their BioProject (PRJNA488366) and SRA runs (SRX4638772-SRX4639021)
- No duplicate ORGANISM identifiers across assembly projects, so downstream BiG-SCAPE reports the full genome count rather than collapsing 103 to 96
- Each genome passes into antiSMASH (--minimal) and appears as a row/column in the genome-by-GCF absence/presence matrix

## Limitations

- BiG-SCAPE groups scaffolds by ORGANISM identifier, so duplicate ORGANISM identifiers across assembly projects silently reduce the reported genome count (96 of 103 Streptomyces in the paper)
- Draft assemblies require an assembly step (SPAdes with default options) before BGC prediction; raw SRA reads are not directly usable
- Some genomes used in the full analysis (CORASON/EvoMining) were unpublished drafts with no NCBI retrieval path, and MS/MS data for non-detoxin-producing strains was only available upon request

## Evidence
