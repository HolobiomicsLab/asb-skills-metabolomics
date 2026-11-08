---
name: cross-dataset-masst-search
description: Use when an annotation made in one full-scan dataset (e.g. a pseudo MS/MS
  spectrum from NIST feces MSV000095787) must be validated or propagated by finding
  the top cosine match in a pool of pseudo MS/MS spectra from another dataset (e.g.
  IBD, PR000639).
when_to_use_negative:
- The data contain real MS/MS scans — standard MS/MS library search (e.g. GNPS library
  via ms1-id reverse matching) is the appropriate route, not cross-dataset pseudo-MS/MS
  matching
- The goal is first-pass structural annotation of a single dataset (use the ms1-id
  lcms/msi workflows) rather than validating or propagating an existing annotation
  across datasets
- The target dataset has no pre-generated pseudo MS/MS pool and in-source-fragment
  clustering cannot be performed
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: Performs the flash-entropy-based forward cosine similarity search of the query
    pseudo MS/MS spectrum against the pool of pseudo MS/MS spectra from the second
    dataset; invoked via its 'annotate' mode on mgf input
  repo: https://github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:e97a9081f4fc67ffb54f9a830bcac212a2d6a263fa2bbde6a910dbb2a64ca2bd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - ms1-id
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cross-dataset-masst-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MASST-style proof-of-principle search that takes one pseudo MS/MS spectrum generated from full-scan MS1 data in one dataset and searches it forward (cosine similarity) against a pool of pseudo MS/MS spectra pre-generated from a different dataset, propagating and validating N-acyl amide-type annotations across cohorts.

## When to use

Use when an annotation made in one full-scan dataset (e.g. a pseudo MS/MS spectrum from NIST feces MSV000095787) must be validated or propagated by finding the top cosine match in a pool of pseudo MS/MS spectra from another dataset (e.g. IBD, PR000639).

## When NOT to use

- The data contain real MS/MS scans — standard MS/MS library search (e.g. GNPS library via ms1-id reverse matching) is the appropriate route, not cross-dataset pseudo-MS/MS matching
- The goal is first-pass structural annotation of a single dataset (use the ms1-id lcms/msi workflows) rather than validating or propagating an existing annotation across datasets
- The target dataset has no pre-generated pseudo MS/MS pool and in-source-fragment clustering cannot be performed

## Inputs

- Query pseudo MS/MS spectrum (mgf) generated from full-scan MS1 data of one dataset, with metadata (precursor mass, annotation)
- Pool of pseudo MS/MS spectra pre-generated from a second full-scan dataset (e.g. IBD HILIC positive cohort, PR000639)
- Indexed MS/MS or pseudo-MS/MS libraries (mgf/msp) compatible with ms1-id flash-entropy search

## Outputs

- Ranked list of cosine matches of the query against every spectrum in the target dataset pool
- Top-hit record: matched precursor mass, cosine score, and propagated annotation (e.g. 'masst_search_result.json')

## How to apply

Load the query pseudo MS/MS spectrum (e.g. phenylalanine-C3:0 generated from NIST feces full-scan data, MSV000095787) and the pre-generated pseudo MS/MS pool of the target dataset (IBD HILIC positive, Metabolomics Workbench PR000639). Run a forward cosine similarity search of the query against every spectrum in the pool using the flash-entropy-based cosine matching implemented in ms1-id, rather than modified-cosine library search. Rank all matches, retain the top hit, and verify it clears the ms1-id default cutoffs (minimum score 0.7, minimum 3 matched peaks) and reaches the expected score (0.90 in the phenylalanine-C3:0 example; molecular-networking edges require modified cosine >= 0.8 with >= 6 matched peaks for propagation). Confirm the matched precursor mass and annotation (e.g. Tyr-C8:1-like N-acyl amides showing tyrosine fragmentation and neutral loss of the fatty acyl chain) and, if a biological claim is made, test abundance differences with Mann-Whitney U tests plus IQR outlier removal (IBD missing values filled with the minimum of 5E5 and 10% of minimum feature intensity). Write the query metadata, top-hit precursor mass, cosine score, and annotation to a results file such as 'masst_search_result.json'.

## Related tools

- **ms1-id** (Performs the flash-entropy-based forward cosine similarity search of the query pseudo MS/MS spectrum against the pool of pseudo MS/MS spectra from the second dataset; invoked via its 'annotate' mode on mgf input) — https://github.com/Philipbear/ms1_id

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- Top hit attains a cosine score above the ms1-id cutoff of 0.7; in the phenylalanine-C3:0 vs IBD example the expected score is 0.90
- At least 3 matched peaks between query and top-hit spectra (ms1-id minimum matched peaks = 3); >= 6 matched peaks with modified cosine >= 0.8 if used to build a propagation/network edge
- Matched precursor mass is consistent with the query feature (precursor-tolerant matching must not have drifted to an unrelated mass)
- Result JSON contains query spectrum metadata, top-hit precursor mass, cosine score, and annotation (e.g. phenylalanine-C3:0)
- Propagated annotation is corroborated by neighbor-node delta masses and mirror-plot fragment patterns (tyrosine fragmentation plus neutral loss of the fatty acyl chain)

## Limitations

- Matching quality depends on peak intensity scaling, which is needed because low-energy in-source fragments must be matched against reference spectra collected at medium to high collision energies
- Isotope peaks (determined by MassCube) must be excluded from pseudo MS/MS generation, and for networking all ions larger than the feature m/z are removed before modified cosine calculations
- Cross-dataset propagation is spectral-similarity-based and at most supports level 2/3 annotations; the reverse search score and mass-dependent peak scaling equations are not fully rendered in the article text

## Evidence

- [results] we demonstrated that this peak scaling approach provided more matching scores of >0.7 for ground truths compared to not applying peak scaling: "we demonstrated that this peak scaling approach provided more matching scores of >0.7 for ground truths compared to not applying peak scaling"
- [discussion] A minimum of modified cosine of 0.8 and matched peaks of 6 are required to build an edge in the network construction: "A minimum of modified cosine of 0.8 and matched peaks of 6 are required to build an edge in the network construction"
- [readme] ms1_id is a Python package that annotates full-scan MS data using tandem MS libraries, specifically: annotate pseudo MS/MS spectra: mgf files: [evidence span withheld — anchor status: not_found]
