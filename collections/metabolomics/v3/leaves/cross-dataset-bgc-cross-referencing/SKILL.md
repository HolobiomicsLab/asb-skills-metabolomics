---
name: cross-dataset-bgc-cross-referencing
description: Use when mass spectrometry identifications from multiple GNPS microbial
  datasets can be paired with genome mining data from the same strains and you need
  to validate identifications or discover novel BGC-to-molecule links.
when_to_use_negative:
- The sample spectra lack paired genomic data for the same strains, so no metabologenomic
  linkage is possible
- The goal is plain spectral library search or de novo structure prediction rather
  than genome-molecule cross-referencing
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3174
tools:
- name: molDiscovery
  role: Identifies small molecules in the microbial datasets whose products are then
    cross-referenced against genome-mined BGCs
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:2624ca2bd85e74dd31b35ab639e2775e8c5eaf1400278be9f062ef7ffe2895dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Cross-dataset BGC cross-referencing

## Summary

Metabologenomic validation of mass-spectrometry identifications by linking molecules found by molDiscovery to biosynthetic gene clusters (BGCs) in genomes of the same strains, distinguishing known molecule-BGC pairs from putative novel BGC families.

## When to use

Use when mass spectrometry identifications from multiple GNPS microbial datasets can be paired with genome mining data from the same strains and you need to validate identifications or discover novel BGC-to-molecule links.

## When NOT to use

- The sample spectra lack paired genomic data for the same strains, so no metabologenomic linkage is possible
- The goal is plain spectral library search or de novo structure prediction rather than genome-molecule cross-referencing

## Inputs

- molDiscovery identification results for all target GNPS microbial datasets (mass spectra search hits)
- MIBiG-linked BGC annotations from genomes of the corresponding microbial strains

## Outputs

- Molecule-to-BGC linkage table (bgc_molecule_links.tsv) with accepted links meeting coverage >= 90% and identity >= 70%
- Count of known molecule-BGC links and putative novel BGC families

## How to apply

Load molDiscovery identification results for all microbial datasets (e.g., the 46 GNPS microbial datasets covering 922 strains) and the MIBiG-linked BGC annotations from the corresponding strain genomes. For each identified molecule with a linked gene cluster, run BLAST comparisons against MIBiG reference clusters to obtain alignment coverage and percent identity. Accept a molecule-to-BGC link only when BLAST alignment coverage is >= 90% and identity is >= 70%. In the reference application this workflow recovered 19 known molecule-BGC links (xantholysin A/B/C, orfamide A/B/C, hygromycin A, yersiniabactin) and 3 putative novel BGC families (dinghupeptins, lipopeptin/neopeptin, longicatenamycin); the accepted links are compiled into a linkage table for inspection.

## Related tools

- **molDiscovery** (Identifies small molecules in the microbial datasets whose products are then cross-referenced against genome-mined BGCs) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- The linkage table contains exactly 19 known and 3 putative BGC-to-molecule links as reported in the reference study
- Every accepted link satisfies both BLAST criteria: alignment coverage >= 90% and percent identity >= 70%
- The recovered known links include the expected examples: xantholysin A/B/C, orfamide A/B/C, hygromycin A, and yersiniabactin
- The putative novel families recovered are dinghupeptin, lipopeptin/neopeptin, and longicatenamycin

## Limitations

- Only datasets with paired mass spectrometry and genomic data can be cross-referenced
- Identifications underlying the links depend on molDiscovery accuracy, which is lower in the low-mass range (<600 Da) because fewer fragments are generated for small molecules

## Evidence

- [intro] BGC linkage validation on genomic datasets: "molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products"
- [abstract] Identification performance: "molDiscovery correctly identify six times more unique small molecules than previous methods."
- [intro] Scope of input spectra: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [readme] Tool access point: "To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/"
