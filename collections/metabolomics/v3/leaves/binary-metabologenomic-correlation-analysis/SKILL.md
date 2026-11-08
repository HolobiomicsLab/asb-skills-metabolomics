---
name: binary-metabologenomic-correlation-analysis
description: Use when you have a GCF presence/absence matrix derived from BGC clustering
  and an ion presence/absence table from MS data, and need to link gene cluster families
  to candidate molecular features.
when_to_use_negative:
- When GCFs are already matched to molecular families based on observed/predicted
  chemical features rather than requiring statistical correlation
- When GCFs or ions are detected in fewer than two strains — non-correlatable GCFs
  and singletons must be excluded before running correlations
- When the input is raw MS/MS spectra rather than a processed binary ion presence/absence
  table
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: BiG-SCAPE
  role: Constructs BGC sequence similarity networks and assigns GCFs via network filtering
    at distance cutoffs plus Affinity Propagation clustering, producing the GCF presence/absence
    matrix
  repo: https://git.wageningenur.nl/medema-group/BiG-SCAPE
- name: antiSMASH v4
  role: Predicts BGCs and outputs GenBank files that serve as direct inputs to BiG-SCAPE
    (run with --minimal)
- name: HMMER 3.1b2
  role: Detects conserved domains against Pfam version 31 for pairwise BGC distance
    calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-019-0400-9
    title: A computational framework to explore large-scale biosynthetic diversity
  source_content_hash: sha256:b19ecc6f41e11e2f90f89f5967c71f17ce3e03cb9a4e041e586140b67f84dd1c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiG-SCAPE
  - HMMER 3.1b2
  - antiSMASH v4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# binary-metabologenomic-correlation-analysis

## Summary

Correlates binary GCF presence/absence (from BiG-SCAPE network clustering) against binary ion presence/absence tables to connect gene cluster families to molecular features, as done for 363 Streptomyces strains in the BiG-SCAPE/CORASON framework.

## When to use

Use when you have a GCF presence/absence matrix derived from BGC clustering and an ion presence/absence table from MS data, and need to link gene cluster families to candidate molecular features.

## When NOT to use

- When GCFs are already matched to molecular families based on observed/predicted chemical features rather than requiring statistical correlation
- When GCFs or ions are detected in fewer than two strains — non-correlatable GCFs and singletons must be excluded before running correlations
- When the input is raw MS/MS spectra rather than a processed binary ion presence/absence table

## Inputs

- antiSMASH-predicted BGC GenBank files for each strain (363 strains in the reference study)
- Binary ion presence/absence table from MS/MS data (5,824 ions detected in two or more strains)
- Pfam database version 31 with HMMER 3.1b2 for BiG-SCAPE domain detection
- List of known molecule-GCF pairs (9 pairs) used for validation

## Outputs

- Binary GCF presence/absence matrix (363 strains x GCFs) per mode/cutoff combination
- Per-pair correlation scores and default-mode selection summary written to metabologenomic_correlations.csv
- Selected default BiG-SCAPE setting (glocal mode, 0.30 distance cutoff)

## How to apply

Run BiG-SCAPE (HMMER 3.1b2 with Pfam version 31) on antiSMASH-predicted BGC GenBank files under both global and glocal alignment modes, filtering networks at distance cutoffs 0.30 and 0.50 for each mode. Assign GCFs by applying Affinity Propagation clustering on each filtered network and build a binary GCF presence/absence matrix (363 strains x GCFs) per mode/cutoff combination. Restrict the matrix to correlatable GCFs (representative gene clusters in two or more strains) and run binary correlations of GCF presence/absence against the ion presence/absence table (5,824 ions detected in two or more strains, identified by mass within 4 ppm and fragmentation cosine similarity above 0.75). Compare the four mode/cutoff combinations and select glocal/0.30 as the default, validating that at least 6 of 9 known molecule-GCF pairs fall in the rightmost tail of the correlation score distribution.

## Related tools

- **BiG-SCAPE** (Constructs BGC sequence similarity networks and assigns GCFs via network filtering at distance cutoffs plus Affinity Propagation clustering, producing the GCF presence/absence matrix) — https://git.wageningenur.nl/medema-group/BiG-SCAPE
- **antiSMASH v4** (Predicts BGCs and outputs GenBank files that serve as direct inputs to BiG-SCAPE (run with --minimal))
- **HMMER 3.1b2** (Detects conserved domains against Pfam version 31 for pairwise BGC distance calculation)

## Evaluation signals

- With the selected default (glocal/0.30), at least 6 of the 9 known molecule-GCF pairs fall in the rightmost tail of the metabologenomic correlation distribution
- The binary GCF matrix is restricted to correlatable GCFs only (representative gene clusters in ≥2 strains); verify no singletons remain
- The ion table matches the study's filter: 5,824 ions detected in two or more strains, with mass within 4 ppm and fragmentation cosine similarity above 0.75
- All four mode/cutoff combinations (global/glocal x 0.30/0.50) are compared before default selection, not just one
- Output file metabologenomic_correlations.csv contains per-pair correlation scores and a selection summary

## Limitations

- Result depends on BiG-SCAPE mode and cutoff choice; glocal/0.30 was selected empirically by comparison across four combinations
- MS/MS data for non-detoxin-producing strains was not deposited (available only upon request), limiting full reproduction of the metabolomic side
- BiG-SCAPE interactive report covered only 96 of 103 Streptomyces genomes because scaffolds belonging to the same genome are grouped by ORGANISM identifier across assembly projects

## Evidence

- [intro] We confirm that metabologenomic correlations accurately connect GCFs to mass features across metabolomic data from 363 strains: "We confirm that metabologenomic correlations accurately connect GCFs to mass features across metabolomic data from 363 strains"
- [discussion] GCFs that had representative gene clusters in two or more strains were considered correlatable: "GCFs that had representative gene clusters in two or more strains were considered correlatable"
- [discussion] Ion identity requires mass within 4 ppm and cosine similarity above 0.75: "within 4 ppm and fragmentation cosine similarity scores were above 0.75, yielding 5,824 ions detected in two or more strains"
- [discussion] GCF assignment via network filtering plus Affinity Propagation clustering: "BiG-SCAPE creates a network using all distances lower or equal than the current cutoff. The Affinity Propagation clustering algorithm39 is applied"
- [readme] BiG-SCAPE computes pairwise distances from domain content, order, copy number and sequence identity: [evidence span withheld — anchor status: external_doc]
