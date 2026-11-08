---
name: hypergeometric-score-standardisation
description: Use when raw Metcalf/strain correlation scores between BGC clusters (GCFs)
  and molecular families (MFs) must be compared across links of different GCF and
  MF sizes, and raw scores fail to prioritise validated links.
when_to_use_negative:
- When the goal is a de novo link-scoring function rather than rescaling existing
  Metcalf scores (consider the Input-Output Kernel Regression score instead)
- When GCF and MF size distributions are homogeneous and raw scores already rank validated
  links above all links — standardisation adds no information
- When strain-to-genome/metabolome mappings (strain_mappings.json) are unavailable,
  since the hypergeometric null is built from strain co-occurrence vectors
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_1775
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Framework that loads the paired genomics/metabolomics data, builds GCF–MF
    links, and computes metcalf scores via npl.get_links
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Source of molecular networking data (spectra, molecular families) used as
    metabolomic input
- name: AntiSMASH
  role: Source of BGC regions feeding the genomic side of the co-occurrence vectors
- name: BigScape
  role: Clusters AntiSMASH BGCs into GCFs used as one side of the scored links
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:aa05f71c38fd4497bb4cc2bc5b0dead084f0af63b57036287dd94d562eaf76e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - AntiSMASH
  - BigScape
  - GNPS
  - NPLinker
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_1775
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hypergeometric-score-standardisation

## Summary

Standardises raw strain correlation (Metcalf) scores for GCF–MF links against their hypergeometric null expectation (expected score and standard deviation under random strain assignment), removing size-dependent bias so that validated links reliably outscore all links on average.

## When to use

Use when raw Metcalf/strain correlation scores between BGC clusters (GCFs) and molecular families (MFs) must be compared across links of different GCF and MF sizes, and raw scores fail to prioritise validated links.

## When NOT to use

- When the goal is a de novo link-scoring function rather than rescaling existing Metcalf scores (consider the Input-Output Kernel Regression score instead)
- When GCF and MF size distributions are homogeneous and raw scores already rank validated links above all links — standardisation adds no information
- When strain-to-genome/metabolome mappings (strain_mappings.json) are unavailable, since the hypergeometric null is built from strain co-occurrence vectors

## Inputs

- GNPS molecular networking data (GNPS1 or GNPS2 workflow export)
- AntiSMASH BGC regions (antismash directory)
- BiG-SCAPE GCF clustering output (bigscape directory)
- strain_mappings.json associating BGC and spectrum identifiers with strains
- PoDP-validated link set for evaluation

## Outputs

- Per-link raw and standardised Metcalf scores (e.g. crusemann_link_scores.csv)
- Group mean raw and standardised scores for the all-links and validated-links sets

## How to apply

Load the Crüsemann data set (GNPS molecular networking files, AntiSMASH BGC regions and BiG-SCAPE GCF clustering) into NPLinker via load_config and load_data. Read strain_mappings.json to associate each BGC and spectrum identifier with its strain, building strain co-occurrence vectors for GCFs and molecular families. Compute raw strain correlation scores for every hypothetical GCF–MF link with npl.get_links using the metcalf scoring method. For each raw score, compute the expected score and standard deviation under a hypergeometric null (random strain assignment) and convert to the standardised score. Then compare group means: on the Crüsemann data the raw scores give 83.514 for all links vs 14.667 for validated links (validating links score lower), whereas standardised scores give -0.006 for all links vs 3.672 for validated links, reversing the failure mode so validated links rank higher.

## Related tools

- **NPLinker** (Framework that loads the paired genomics/metabolomics data, builds GCF–MF links, and computes metcalf scores via npl.get_links) — https://github.com/NPLinker/nplinker
- **GNPS** (Source of molecular networking data (spectra, molecular families) used as metabolomic input)
- **AntiSMASH** (Source of BGC regions feeding the genomic side of the co-occurrence vectors)
- **BigScape** (Clusters AntiSMASH BGCs into GCFs used as one side of the scored links)

## Examples

```
from nplinker import load_config; npl = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Mean standardised score for PoDP-validated links exceeds the mean for all links (target: 3.672 vs -0.006 on the Crüsemann data set)
- Raw-score baseline reproduces the known failure mode (all links 83.514 vs validated links 14.667 mean Metcalf score) before standardisation
- Per-link CSV contains both raw and standardised score columns for every GCF–MF pair, with standardised scores computed from the hypergeometric expected score and standard deviation
- Group means match Table 1 / Fig 5 of the reference publication

## Limitations

- Standardisation depends on hypergeometric random strain assignment as the null model; links between very small GCFs or MFs may have unstable standardised scores
- Results verified only on publicly available data sets containing validated links; behaviour on unvalidated data is untested
- Requires strain mappings between genomics and metabolomics data; the article notes extract_mappings_ms_filename_spectrum_id may be needed to map mzXML filenames to spectrum ids

## Evidence

- [other] standardised scores give a mean of -0.006 for all links and 3.672 for validated links: [evidence span withheld — anchor status: not_found]
- [methods] metcalf scoring method link computation: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [methods] strain mapping requirement: "The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data."
- [abstract] standardised score introduction: "Based on standardising a commonly used score, we introduce a new, more effective score, and introduce a novel score using an Input-Output Kernel Regression approach."
- [methods] spectrum id mapping caveat: "you may need the function [extract_mappings_ms_filename_spectrum_id][nplinker.metabolomics.utils.extract_mappings_ms_filename_spectrum_id] to extract the mappings from mzXML files to the spectrum ids"
