---
name: metcalf-strain-correlation-scoring
description: Use when linking AntiSMASH/BigScape-derived GCFs to GNPS molecular families
  by strain co-occurrence, or when raw Metcalf scores must be standardised against
  a null to compare links across GCFs and MFs of different sizes.
when_to_use_negative:
- The spectra or genomes cannot be assigned to shared strains — strain co-occurrence
  vectors cannot be built and Metcalf scoring is undefined
- The analysis needs sequence-homology-based prioritisation against known BGCs (e.g.
  MIBiG homology plus IOKR) rather than strain co-occurrence — use the combined/IOKR
  scoring route instead of raw Metcalf alone
- Molecular networking input is not GNPS molecular networking data or the GCF clustering
  has not been produced from AntiSMASH BGC regions
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3398
tools:
- name: NPLinker
  role: Framework that loads genomics/metabolomics data, builds strain co-occurrence
    vectors, computes Metcalf links via npl.get_links and standardises scores
  repo: github.com/NPLinker/nplinker
- name: AntiSMASH
  role: Provides the BGC region data in the antismash directory that underlie GCF
    construction
- name: BigScape
  role: Clusters AntiSMASH BGCs into GCFs; NPLinker can run it automatically if the
    bigscape directory is absent
- name: GNPS
  role: Source of the molecular networking files (spectra and molecular families)
    that NPLinker requires as input
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:0136c10a73149df70ee2519abbacc3805acb654e8e0f9e700fdd4735c367c7cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AntiSMASH
  - BigScape
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3398
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metcalf-strain-correlation-scoring

## Summary

Computes strain co-occurrence correlation (Metcalf-style) scores between gene cluster families (GCFs) and molecular families (MFs) in NPLinker, then standardises each raw score against its hypergeometric null expectation to remove dependence on GCF and MF sizes, enabling fair comparison and prioritisation of GCF–MF links.

## When to use

Use when linking AntiSMASH/BigScape-derived GCFs to GNPS molecular families by strain co-occurrence, or when raw Metcalf scores must be standardised against a null to compare links across GCFs and MFs of different sizes.

## When NOT to use

- The spectra or genomes cannot be assigned to shared strains — strain co-occurrence vectors cannot be built and Metcalf scoring is undefined
- The analysis needs sequence-homology-based prioritisation against known BGCs (e.g. MIBiG homology plus IOKR) rather than strain co-occurrence — use the combined/IOKR scoring route instead of raw Metcalf alone
- Molecular networking input is not GNPS molecular networking data or the GCF clustering has not been produced from AntiSMASH BGC regions

## Inputs

- GNPS molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings.tsv)
- AntiSMASH BGC regions clustered into GCFs by BigScape (versions 1 and 2 supported)
- strain_mappings.json mapping BGC and spectrum identifiers to strains
- NPLinker config file (nplinker.toml)
- PoDP validated GCF–MF links for evaluation

## Outputs

- Per-link raw Metcalf strain correlation scores (e.g. written to crusemann_link_scores.csv)
- Per-link standardised scores (raw score minus hypergeometric null expectation, divided by null standard deviation)
- Ranked candidate GCF–MF link lists, optionally combined ℓ1/ℓ1.5/ℓ2 scores with IOKR (e.g. table4_link_ranks.csv)

## How to apply

Load the data set (GNPS spectra.mgf, molecular_families.tsv, annotations.tsv, antiSMASH-derived BigScape GCFs) into NPLinker via load_config and load_data, then read strain_mappings.json to associate each BGC and spectrum identifier with its strain, building strain co-occurrence vectors for GCFs and MFs. Call npl.get_links on all GCFs with the 'metcalf' scoring method to compute raw strain correlation scores for every hypothetical GCF–MF link. Standardise each raw score against its hypergeometric null expectation (expected score and standard deviation from random strain assignment). On the Crüsemann data set this reverses the raw-score failure mode: all links average 83.514 raw but -0.006 standardised, while PoDP-validated links average 14.667 raw but 3.672 standardised, so validated links score higher than all links only after standardisation. The standardised score can then feed the Eq 2 sign-weighted ℓ1/ℓ1.5/ℓ2 combination with IOKR scores, where ℓ1.5 places the validated link of BGC0000241 (lomaiviticin A) at rank 5 and BGC0001228 (retimycin A) at rank 253.

## Related tools

- **NPLinker** (Framework that loads genomics/metabolomics data, builds strain co-occurrence vectors, computes Metcalf links via npl.get_links and standardises scores) — github.com/NPLinker/nplinker
- **AntiSMASH** (Provides the BGC region data in the antismash directory that underlie GCF construction)
- **BigScape** (Clusters AntiSMASH BGCs into GCFs; NPLinker can run it automatically if the bigscape directory is absent)
- **GNPS** (Source of the molecular networking files (spectra and molecular families) that NPLinker requires as input)

## Examples

```
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- On the Crüsemann data set, group means must match Table 1 / Fig 5: all links 83.514 raw / -0.006 standardised; validated links 14.667 raw / 3.672 standardised
- Standardised scores must reverse the raw-score failure mode: mean standardised score of PoDP-validated links (3.672) exceeds that of all links (-0.006)
- When combined with IOKR via ℓ1.5, the validated link receives the best rank in 10 of 15 validated cases (e.g. BGC0000241 at rank 5, BGC0001228 at rank 253)
- Every link carries both a raw score and a null-derived expected score/standard deviation; links lacking a finite standardised value indicate missing strain mappings
- Reproducibility check: rerunning get_links with the 'metcalf' method on the same strain_mappings.json yields identical per-link scores in the output CSV

## Limitations

- The raw Metcalf score depends on GCF and MF sizes, so raw scores fail to reliably compare potential GCF–MF links; standardisation is required before cross-link comparison
- Evaluation depends on the availability of PoDP-validated links, which are limited (e.g. 15 validated cases in Table 4)
- Requires strain_mappings.json strain assignments; the paired Omics Data Platform (PODP) is used as the input for NPLinker to supply these mappings

## Evidence

- [methods] NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data.: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data."
- [methods] strain_mappings.json: "The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data."
- [methods] metcalf scoring method: "# compute the links for the first 3 GCFs using metcalf scoring method"
- [abstract] standardised score abstract: "Based on standardising a commonly used score, we introduce a new, more effective score, and introduce a novel score using an Input-Output Kernel Regression approach."
- [readme] NPLinker can run BigScape: "NPLinker can run BigScape automatically if the `bigscape` directory does not exist in the working directory. Both version 1 and version 2 of BigScape are supported."
