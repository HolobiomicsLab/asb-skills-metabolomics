---
name: biosynthetic-gene-cluster-prediction-antismash
description: Use when starting from complete or draft genome assemblies (GenBank/WGS
  files) and BGC predictions are needed as the entry point for a comparative biosynthesis
  workflow such as BiG-SCAPE GCF clustering.
when_to_use_negative:
- Input is already antiSMASH GenBank output — proceed directly to BiG-SCAPE clustering
  instead of re-running prediction
- The goal is GCF phylogenetic ordering rather than de novo BGC detection — use CORASON
  on existing antiSMASH outputs instead
- Genomes are highly fragmented metagenomic assemblies where the study itself notes
  current methods fail to measure similarity between complete and fragmented clusters
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3174
tools:
- name: antiSMASH v4
  role: Predicts BGCs from genome files using the --minimal parameter setting; its
    GenBank outputs are the direct input format for BiG-SCAPE
- name: BiG-SCAPE
  role: Consumes antiSMASH GenBank outputs to build sequence similarity networks and
    cluster BGCs into GCFs
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: MIBiG
  role: Reference BGC database (release 1.3) whose GenBank records can be co-analyzed
    with antiSMASH predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:b9f20a2fc794bca3d96a1bd9620786f894d88bda35f6660efccafd1a01a8fe40
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - MIBiG
  - antiSMASH v4
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# biosynthetic-gene-cluster-prediction-antismash

## Summary

Predicts biosynthetic gene clusters (BGCs) from genome sequences with antiSMASH, producing antiSMASH GenBank outputs that serve directly as inputs for downstream BiG-SCAPE/CORASON sequence similarity networking and gene cluster family (GCF) assembly.

## When to use

Use when starting from complete or draft genome assemblies (GenBank/WGS files) and BGC predictions are needed as the entry point for a comparative biosynthesis workflow such as BiG-SCAPE GCF clustering.

## When NOT to use

- Input is already antiSMASH GenBank output — proceed directly to BiG-SCAPE clustering instead of re-running prediction
- The goal is GCF phylogenetic ordering rather than de novo BGC detection — use CORASON on existing antiSMASH outputs instead
- Genomes are highly fragmented metagenomic assemblies where the study itself notes current methods fail to measure similarity between complete and fragmented clusters

## Inputs

- Complete or draft genome sequences in GenBank format (e.g. NCBI WGS/complete genome records such as CP001700, CP000850)
- Optionally MIBiG reference BGC GenBank records (e.g. MIBiG release 1.3 entries such as BGC0001840)

## Outputs

- antiSMASH GenBank files containing predicted BGCs per genome, with genes annotated as 'Core Biosynthetic' or otherwise
- BGC predictions ready to be used directly as BiG-SCAPE inputs

## How to apply

Run antiSMASH (the reference study used version 4 with the --minimal parameter setting) on each downloaded genome to predict BGCs and annotate each gene's role, including which genes antiSMASH marks as 'Core Biosynthetic' — this annotation is later required by BiG-SCAPE's LCS filtering rule. Collect the antiSMASH GenBank output files per genome, optionally including MIBiG reference BGCs alongside them, because these GenBank outputs can be used directly as inputs for the BiG-SCAPE/CORASON workflow without reformatting. In the 103-Streomyces-genome example, complete genomes plus outgroups (Catenulispora acidiphila CP001700, Salinispora arenicola CP000850) were retrieved from NCBI before processing, and the antiSMASH outputs fed into BiG-SCAPE with Pfam version 31 searched by HMMER 3.1b2 to yield the BGCs grouped into 27 conserved and 1,564 rare GCFs. Judging correctness: antiSMASH completes on every input genome, emits valid GenBank BGC records, and the gene marked 'Core Biosynthetic' is present in the outputs so downstream clustering can proceed.

## Related tools

- **antiSMASH v4** (Predicts BGCs from genome files using the --minimal parameter setting; its GenBank outputs are the direct input format for BiG-SCAPE)
- **BiG-SCAPE** (Consumes antiSMASH GenBank outputs to build sequence similarity networks and cluster BGCs into GCFs) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **MIBiG** (Reference BGC database (release 1.3) whose GenBank records can be co-analyzed with antiSMASH predictions)

## Examples

```
run_antismash --minimal genome.gbk
```

## Evaluation signals

- Every input genome yields an antiSMASH output directory with valid BGC GenBank records
- Predicted BGC records contain the antiSMASH 'Core Biosynthetic' gene annotations required by BiG-SCAPE's downstream LCS rule ('the LCS must be either three genes long, or contain at least one gene marked by antiSMASH as Core Biosynthetic')
- When the 103-genome Streptomyces example is followed, downstream BiG-SCAPE clustering with Pfam 31/HMMER 3.1b2 and cutoff 0.3 should reproduce 27 highly conserved GCFs (~2%, >10 genomes) and 1,564 rare GCFs (92%, ≤3 genomes)
- antiSMASH version and parameters are recorded for reproducibility (v4, --minimal), matching the published workflow

## Limitations

- Fragmented BGCs (common in metagenomes and large-scale pan-genome projects) complicate downstream similarity measurement; the paper states 'current methods fail to correctly measure the similarity between complete and fragmented gene clusters'
- BiG-SCAPE's interactive report grouped scaffolds by ORGANISM identifier, reporting only 96 of 103 Streptomyces genomes when duplicate ORGANISM identifiers spanned assembly projects
- Conservative --minimal parameter setting was chosen; it may omit some detection detail relative to full antiSMASH runs

## Evidence
