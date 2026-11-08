---
name: gcf-presence-absence-matrix-construction
description: Use when antiSMASH GenBank BGC predictions across many strains must be
  converted into a binary GCF presence/absence table (e.g., 363 strains x GCFs) as
  input for metabologenomic correlation against MS-detected ions.
when_to_use_negative:
- The input is already a presence/absence or quantitative feature table (e.g. the
  5,824-ion table detected in two or more strains) — go straight to correlation instead.
- The goal is high-resolution phylogenetic ordering of gene clusters within a single
  GCF (use CORASON multi-locus phylogenies rather than presence/absence binning).
- GCFs must be matched to molecular families by shared chemical features from MS/MS
  data rather than by binary correlation across strains.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SCAPE
  role: Calculates pairwise BGC distances from Pfam domain content, builds the filtered
    sequence similarity network, and applies Affinity Propagation to assign GCFs for
    the presence/absence matrix
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH v4
  role: Produces the BGC GenBank predictions (--minimal parameters) used as direct
    inputs to BiG-SCAPE
- name: HMMER 3.1b2
  role: Detects conserved domains against the user-supplied Pfam database version
    31 during BiG-SCAPE processing; overlapping domains are filtered by hmmer score
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:9c61cf0fce2cdcbe54b2f9bfd288d6882873235955fa015033c2ad0a4dc7d6f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# gcf-presence-absence-matrix-construction

## Summary

Constructs a binary gene cluster family (GCF) presence/absence matrix by running BiG-SCAPE on antiSMASH-predicted BGC GenBank files, filtering the sequence similarity network at a distance cutoff, and assigning GCFs via Affinity Propagation. This matrix enables metabologenomic correlation of genomic potential against ion presence/absence tables.

## When to use

Use when antiSMASH GenBank BGC predictions across many strains must be converted into a binary GCF presence/absence table (e.g., 363 strains x GCFs) as input for metabologenomic correlation against MS-detected ions.

## When NOT to use

- The input is already a presence/absence or quantitative feature table (e.g. the 5,824-ion table detected in two or more strains) — go straight to correlation instead.
- The goal is high-resolution phylogenetic ordering of gene clusters within a single GCF (use CORASON multi-locus phylogenies rather than presence/absence binning).
- GCFs must be matched to molecular families by shared chemical features from MS/MS data rather than by binary correlation across strains.

## Inputs

- antiSMASH-predicted BGC GenBank files for each strain (e.g. from 363 strains, antiSMASH v4 run with --minimal)
- Reference BGC GenBank files from MIBiG (release 1.3)
- Pfam database version 31 for HMMER domain detection
- BiG-SCAPE mode/cutoff parameters (global vs glocal; distance cutoffs 0.30 and 0.50)

## Outputs

- Binary GCF presence/absence matrix (363 strains x GCFs) per mode/cutoff combination
- Correlatable-GCF-restricted matrix (GCFs with representatives in two or more strains)
- Per-pair metabologenomic correlation scores and mode/cutoff selection summary (e.g. metabologenomic_correlations.csv)

## How to apply

First run BiG-SCAPE (with HMMER 3.1b2 against the Pfam database version 31) on the antiSMASH-predicted BGC GenBank files; the paper evaluated both global and glocal alignment modes, generating networks filtered at distance cutoffs 0.30 and 0.50, and selected glocal/0.30 as the default. BiG-SCAPE filters the network to all pairwise distances lower or equal than the cutoff and then applies the Affinity Propagation clustering algorithm to assign gene clusters to GCFs. From the resulting GCF assignments, build a binary presence/absence matrix (rows = strains, columns = GCFs). Before downstream correlation, restrict the matrix to correlatable GCFs, i.e. those with representative gene clusters in two or more strains; a GCF present in only one strain carries no correlation signal. Validate the matrix by confirming that known molecule-GCF pairs (e.g. detoxin and rimosamide families) fall in the rightmost tail of the metabologenomic correlation distribution under the chosen mode/cutoff combination.

## Related tools

- **BiG-SCAPE** (Calculates pairwise BGC distances from Pfam domain content, builds the filtered sequence similarity network, and applies Affinity Propagation to assign GCFs for the presence/absence matrix) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH v4** (Produces the BGC GenBank predictions (--minimal parameters) used as direct inputs to BiG-SCAPE)
- **HMMER 3.1b2** (Detects conserved domains against the user-supplied Pfam database version 31 during BiG-SCAPE processing; overlapping domains are filtered by hmmer score)

## Examples

```
python bigscape.py -i antismash_gbk/ -o out_mg --mode glocal -c 0.30
```

## Evaluation signals

- Matrix dimensions match the strain panel (363 strains) with one column per Affinity Propagation-assigned GCF and strictly binary (0/1) entries.
- Every GCF retained for correlation has representative gene clusters in two or more strains; singletons are excluded.
- Under the selected glocal/0.30 setting, at least 6 of the 9 known molecule-GCF combinations fall in the rightmost tail of the metabologenomic correlation distribution.
- GCF calling is stable across domain-alignment backends: comparison using BiG-SCAPE's --mix parameter showed high agreement between hmmalign, Muscle, and MAFFT, with hmmalign shortest at runtime for large BGC sets.
- Cross-check the mode/cutoff choice against all four combinations (global/glocal x 0.30/0.50) before locking in the matrix used for correlation.

## Limitations

- Duplicate ORGANISM identifiers can collapse genomes in BiG-SCAPE's interactive report (only 96 of 103 Streptomyces genomes reported because scaffolds from the same genome are grouped by ORGANISM identifier); verify strain counts downstream.
- The GCF assignment depends on the network cutoff; the paper reports 0.3 as a predefined empirical cutoff, and the mode/cutoff combination was selected by comparing correlation performance rather than from first principles.
- GCFs restricted to a single strain are non-correlatable and are dropped, so rare pathways will be invisible in correlation-based analyses.

## Evidence

- [readme] BiG-SCAPE role in constructing BGC similarity networks and grouping GCFs: [evidence span withheld — anchor status: external_doc]
