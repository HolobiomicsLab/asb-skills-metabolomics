---
name: validated-link-rank-evaluation
description: Use when a GCF with established homology to a MIBiG BGC (e.g. BGC0001228,
  BGC0000241) must be scored against all molecular families, and the validated link's
  rank under each scoring function is needed to judge which score prioritises true
  links .
when_to_use_negative:
- The GCF of interest has no established homology to a validated MIBiG BGC, so there
  is no ground-truth link to rank against.
- The goal is de novo prioritisation of all GCF–molecular-family links rather than
  evaluation of known validated links.
- Genomics or metabolomics data lack strain-level mapping (no strain_mappings.json),
  so standardised strain-correlation scores cannot be computed.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: nplinker
  role: Loads the Crüsemann data set and computes GCF–molecular-family links with
    npl.get_links using the metcalf scoring method
  repo: https://github.com/NPLinker/nplinker
- name: AntiSMASH
  role: Supplies the BGC data from which BigScape GCFs are derived
- name: BigScape
  role: Clusters antiSMASH BGCs into the GCFs scored against molecular families
- name: GNPS
  role: Provides the molecular networking data (spectra.mgf, molecular_families.tsv,
    annotations.tsv, file_mappings.tsv) that NPLinker consumes
- name: MIBiG
  role: Supplies the metadata for the validated BGCs (e.g. BGC0001228, BGC0000241)
    used as ground-truth links
- name: Python
  role: Runtime for computing standardised and IOKR scores and combining them into
    ℓp scores
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:fbc52bd8e89dbbbc6e7a2ea7570a813956c27df6011fa2996631280db82eeb83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - AntiSMASH
  - BigScape
  - GNPS
  - MIBiG
  - Python
  - nplinker
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# validated-link-rank-evaluation

## Summary

Ranks validated GCF–molecular-family links under combined ℓ1, ℓ2, and ℓ1.5 scores built from standardised strain-correlation (metcalf) and IOKR scores, and compiles the rank counts into a Table 4-style comparison for the Crüsemann data set.

## When to use

Use when a GCF with established homology to a MIBiG BGC (e.g. BGC0001228, BGC0000241) must be scored against all molecular families, and the validated link's rank under each scoring function is needed to judge which score prioritises true links best.

## When NOT to use

- The GCF of interest has no established homology to a validated MIBiG BGC, so there is no ground-truth link to rank against.
- The goal is de novo prioritisation of all GCF–molecular-family links rather than evaluation of known validated links.
- Genomics or metabolomics data lack strain-level mapping (no strain_mappings.json), so standardised strain-correlation scores cannot be computed.

## Inputs

- GNPS spectra.mgf file
- molecular_families.tsv from GNPS molecular networking
- annotations.tsv from GNPS molecular networking
- file_mappings.tsv from GNPS molecular networking
- antiSMASH-derived BGC data clustered into BigScape GCFs
- strain_mappings.json linking BGCs and spectra to strains
- list of validated MIBiG BGC accessions (e.g. BGC0001228, BGC0000241)

## Outputs

- table4_link_ranks.csv with per-validated-link rank counts under ℓ1, ℓ2, and ℓ1.5 combined scores and the best ℓ1.5 rank
- per-GCF–molecular-family-pair standardised metcalf scores and IOKR scores

## How to apply

Load the Crüsemann data set (GNPS spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings.tsv, and antiSMASH-derived BigScape GCFs) into NPLinker via npl.load_data(), connecting BGCs and spectra to strains with strain_mappings.json. Identify the GCFs homologous to the validated MIBiG BGCs and compute links to all molecular families with npl.get_links using the metcalf scoring method as the base correlation score. Standardise the correlation scores, compute Input-Output Kernel Regression (IOKR) scores for all GCF–molecular-family pairs, and combine them into ℓ1, ℓ2, and ℓ1.5 scores via the Eq 2 sign-weighted sum. For each validated link, count candidate links whose combined score is equal to or higher than the validated link's score, record the best rank achieved under the ℓ1.5 combination, and compile the counts and best ranks into table4_link_ranks.csv. Rationale: combining multiple link-scoring functions together makes it easier to prioritise true links relative to others — the combined ℓ1.5 score assigns the best rank in 10 out of 15 cases (e.g. BGC0001228/retimycin A at rank 253, BGC0000241/lomaiviticin A at rank 5).

## Related tools

- **nplinker** (Loads the Crüsemann data set and computes GCF–molecular-family links with npl.get_links using the metcalf scoring method) — https://github.com/NPLinker/nplinker
- **AntiSMASH** (Supplies the BGC data from which BigScape GCFs are derived)
- **BigScape** (Clusters antiSMASH BGCs into the GCFs scored against molecular families)
- **GNPS** (Provides the molecular networking data (spectra.mgf, molecular_families.tsv, annotations.tsv, file_mappings.tsv) that NPLinker consumes)
- **MIBiG** (Supplies the metadata for the validated BGCs (e.g. BGC0001228, BGC0000241) used as ground-truth links)
- **Python** (Runtime for computing standardised and IOKR scores and combining them into ℓp scores)

## Examples

```
npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- The combined ℓ1.5 score assigns the best rank to the validated link in 10 out of 15 cases, matching the published Table 4 result.
- The validated link for BGC0001228 (retimycin A) ranks at number 253 and for BGC0000241 (lomaiviticin A) at number 5 under the combined score.
- Each combined score (ℓ1, ℓ2, ℓ1.5) is computed as the Eq 2 sign-weighted sum of the standardised correlation score and the IOKR score for every GCF–molecular-family pair.
- Rank counts are computed as the number of candidate links whose combined score is equal to or higher than the validated link's score.
- Output table4_link_ranks.csv reproduces the layout of Table 4 with counts and best ranks per validated link.

## Limitations

- Ranks depend on the base metcalf correlation score and standardisation; the ℓ1.5 combination was best in only 10 of 15 validated cases, not all.
- Evaluation is limited to publicly available data sets that include validated links; the count of validated links is small (15 cases).
- NPLinker requires GNPS molecular networking data as input and only accepts GNPS1/GNPS2 workflow output; other metabolomics formats are not covered here.

## Evidence

- [abstract] combined scoring of validated links: "We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others."
- [methods] metcalf link computation: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [abstract] IOKR scoring: "Based on standardising a commonly used score, we introduce a new, more effective score, and introduce a novel score using an Input-Output Kernel Regression approach."
- [methods] strain mapping input: "The strain mappings file `strain_mapping.json` is required by NPLinker to map the strain to genomics and metabolomics data."
- [methods] GNPS input requirement: "NPLinker requires GNPS molecular networking data as input. It currently accepts data from both GNPS1 (https://gnps.ucsd.edu) and GNPS2 (https://gnps2.org) workflows."
