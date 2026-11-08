---
name: gcf-molecular-family-link-scoring
description: Use when NPLinker GCF–molecular-family candidate links must be ranked
  so that PoDP-validated links rise above all-links background, and raw Metcalf scores
  alone are unreliable because of GCF/MF size dependence.
when_to_use_negative:
- The goal is to characterise the raw-score failure mode itself (raw vs standardised
  group means) rather than to rank links with combined scores
- No strain mappings are available, so neither the hypergeometric null standardisation
  nor IOKR strain-based scores can be computed
- No MIBiG homologs or validated links exist for the GCFs of interest, so best-rank
  comparison in Table 4 style cannot be performed
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Framework that loads the Crüsemann data set, computes metcalf-based link data
    via npl.get_links, and hosts the scoring/standardisation pipeline
  repo: https://github.com/NPLinker/nplinker
- name: Python
  role: Runtime for the NPLinker workflow and score combination (Eq 2 sign-weighted
    sum) and rank counting
- name: BigScape
  role: Provides GCF clustering of antiSMASH BGC data when not pre-supplied; NPLinker
    can run it automatically
- name: GNPS
  role: Source of molecular networking input data (GNPS1 or GNPS2 workflows) defining
    molecular families
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:acdfe2175d4d349f486473dff3bffe037955db13d7ccc497d341f0fbe54d2732
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BigScape
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gcf-molecular-family-link-scoring

## Summary

Scores and ranks candidate GCF–molecular-family links in the NPLinker framework by combining a standardised strain correlation score (standardised Metcalf) with an Input-Output Kernel Regression (IOKR) score, using an ℓp sign-weighted sum (ℓ1, ℓ2, ℓ1.5) to prioritise validated links.

## When to use

Use when NPLinker GCF–molecular-family candidate links must be ranked so that PoDP-validated links rise above all-links background, and raw Metcalf scores alone are unreliable because of GCF/MF size dependence.

## When NOT to use

- The goal is to characterise the raw-score failure mode itself (raw vs standardised group means) rather than to rank links with combined scores
- No strain mappings are available, so neither the hypergeometric null standardisation nor IOKR strain-based scores can be computed
- No MIBiG homologs or validated links exist for the GCFs of interest, so best-rank comparison in Table 4 style cannot be performed

## Inputs

- GNPS molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings.tsv)
- antiSMASH BGC data
- BigScape GCF clustering output
- strain_mappings.json mapping BGC and spectrum identifiers to strains
- MIBiG metadata for validated BGCs (e.g. BGC0001228 retimycin A, BGC0000241 lomaiviticin A)

## Outputs

- Raw and standardised strain correlation scores for all GCF–molecular-family links
- IOKR scores for all GCF–molecular-family pairs
- Combined ℓ1, ℓ2 and ℓ1.5 scores per candidate link
- Rank counts of candidate links scoring equal to or above each validated link
- table4_link_ranks.csv in the layout of Table 4

## How to apply

Load the Crüsemann data set (GNPS spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings.tsv, antiSMASH-derived BigScape GCFs) via npl.load_data(), using strain_mappings.json to connect BGCs and spectra to strains. Identify the GCFs homologous to validated MIBiG BGCs (e.g. BGC0001228, BGC0000241) and compute link data between these GCFs and all molecular families with npl.get_links using the metcalf scoring method. Standardise each raw correlation score against its hypergeometric null (expected score and standard deviation under random strain assignment) and compute IOKR scores for all GCF–molecular-family pairs. Combine the standardised scores into ℓ1, ℓ2 and ℓ1.5 combined scores via the Eq 2 sign-weighted sum; the ℓ1.5 combination assigns the best rank to the validated link in 10 of 15 cases (e.g. rank 5 for lomaiviticin A, BGC0000241). For each validated link, count candidate links whose combined score is equal to or higher than the validated link's score and record the best rank achieved under ℓ1.5, compiling results into table4_link_ranks.csv in the layout of Table 4.

## Related tools

- **NPLinker** (Framework that loads the Crüsemann data set, computes metcalf-based link data via npl.get_links, and hosts the scoring/standardisation pipeline) — https://github.com/NPLinker/nplinker
- **Python** (Runtime for the NPLinker workflow and score combination (Eq 2 sign-weighted sum) and rank counting)
- **BigScape** (Provides GCF clustering of antiSMASH BGC data when not pre-supplied; NPLinker can run it automatically)
- **GNPS** (Source of molecular networking input data (GNPS1 or GNPS2 workflows) defining molecular families)

## Examples

```
npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- The ℓ1.5 combined score assigns the best rank to the validated link in 10 out of 15 validated cases (Table 4)
- Validated-link ranks match reported values, e.g. rank 253 for BGC0001228 (retimycin A) and rank 5 for BGC0000241 (lomaiviticin A)
- Standardised-score sanity check: mean standardised score ≈ -0.006 for all links vs 3.672 for validated links (raw means 83.514 vs 14.667)
- table4_link_ranks.csv reproduces the Table 4 layout with per-link candidate counts equal to or above the validated link's combined score
- Each raw score standardises against the hypergeometric null expectation (expected score and standard deviation from random strain assignment)

## Limitations

- The ℓ1.5 combination wins in only 10 of 15 validated cases — combined scoring improves but does not always beat individual scoring functions
- Raw strain correlation scores depend on GCF and molecular-family sizes, so unstandardised scores are not comparable across links
- Assessment depends on the PoDP-validated link set; the discussion section contains no discussion of limitations (only an auto-generated changelog)

## Evidence

- [abstract] We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others.: "We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others."
- [abstract] Based on standardising a commonly used score, we introduce a new, more effective score, and introduce a novel score using an Input-Output Kernel Regression approach.: "Based on standardising a commonly used score, we introduce a new, more effective score, and introduce a novel score using an Input-Output Kernel Regression approach."
- [methods] NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data.: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data."
- [methods] The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data.: "The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data."
- [methods] If the output of BigScape is not provided, NPLinker will run BigScape automatically to generate the data using the AntiSMASH BGC data.: "If the output of BigScape is not provided, NPLinker will run BigScape automatically to generate the data using the AntiSMASH BGC data."
