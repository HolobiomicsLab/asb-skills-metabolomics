---
name: profiler-metric-tie-verification
description: Use when two or more taxonomic profilers report an identical benchmarking
  score (e.g., UniFrac) on a sample and you need to determine whether the underlying
  taxonomic profiles actually differ at specific clades.
when_to_use_negative:
- When a ground truth (gold standard) profile is not available — TAMPA's tie/difference
  verification against a reference requires one; it can only suggest appropriate profilers
  otherwise
- When you need a binning (not profiling) assessment — use AMBER for binning; this
  skill targets taxonomic profile comparisons
- When the profilers' scores are not tied and single-value metrics already separate
  them — a full TAMPA-style clade-level diff may be unnecessary
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxa and visualizes clade-level
    differences between tied profilers and the gold standard that single-value metrics
    miss
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA is integrated for
    metric computation
  repo: https://github.com/CAMI-challenge/OPAL
- name: Taxy-Pro
  role: One of the two tied profilers whose profile is compared against the gold standard
    and the other tied profiler
- name: MetaPhyler
  role: The other tied profiler whose profile is compared against the gold standard
    and Taxy-Pro
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:58d14bb74ae5c494d95a6c9b208681ff00d25ef5b2dff187f3a38aceeea291be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - MetaPhyler
  - OPAL
  - TAMPA
  - Taxy-Pro
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# profiler-metric-tie-verification

## Summary

Verifies whether taxonomic profilers with identical single-value metric scores (e.g., tied UniFrac or F1 scores on a CAMI sample) truly produce identical taxonomic profiles, by computing per-taxon percentage relative abundances and revealing clade-level differences that single-value metrics miss. Grounded in the TAMPA case study where Taxy-Pro and MetaPhyler tied on UniFrac for CAMI_HIGH_S001.

## When to use

Use when two or more taxonomic profilers report an identical benchmarking score (e.g., UniFrac) on a sample and you need to determine whether the underlying taxonomic profiles actually differ at specific clades.

## When NOT to use

- When a ground truth (gold standard) profile is not available — TAMPA's tie/difference verification against a reference requires one; it can only suggest appropriate profilers otherwise
- When you need a binning (not profiling) assessment — use AMBER for binning; this skill targets taxonomic profile comparisons
- When the profilers' scores are not tied and single-value metrics already separate them — a full TAMPA-style clade-level diff may be unnecessary

## Inputs

- Taxonomic profiler submission profile files (Taxy-Pro, MetaPhyler) from CAMI-challenge/firstchallenge_evaluation profiling/data/profile_submissions
- Gold-standard (ground truth) taxonomic profile file for the sample (e.g., CAMI_HIGH_S001)
- description.property file from each profiler's submission subdirectory (to map anonymous submission IDs to tool names)
- NCBI taxdump database (to compute percentage abundance per taxon)

## Outputs

- Computed single-value metric values (e.g., UniFrac distances) for each profiler vs. the gold standard
- Per-taxon relative abundance differences between the tied profilers' profiles
- Tied-metric comparison record (unifrac_comparison.csv) with entries for both profilers

## How to apply

Download the profiler submission and gold-standard profile files from the CAMI first challenge evaluation repo (profiling/data/profile_submissions) and map anonymous submission names to tool names via each subdirectory's description.property file. Build percentage abundance predictions per taxon from the profile files using the NCBI taxdump database, as done by TAMPA. Compute the single-value metric (e.g., UniFrac distance) of each profiler's profile against the gold standard for the sample of interest (e.g., CAMI_HIGH_S001) and confirm the tied values are equal. Then record the per-taxon relative abundance differences between the tied profilers' profiles — TAMPA reveals these at e.g. the phylum level even though single-value metrics report no differences. Write the tied-score verification and abundance differences into a comparison record (e.g., unifrac_comparison.csv); the record is valid only if both computed metric values are identical to the reported tie.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxa and visualizes clade-level differences between tied profilers and the gold standard that single-value metrics miss) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA is integrated for metric computation) — https://github.com/CAMI-challenge/OPAL
- **Taxy-Pro** (One of the two tied profilers whose profile is compared against the gold standard and the other tied profiler)
- **MetaPhyler** (The other tied profiler whose profile is compared against the gold standard and Taxy-Pro)

## Examples

```
python src/tampa.py -i data/metaphyler.profile.txt -i1 data/taxypro.profile.txt -g data/gs_camihigh.profile.txt class -s CAMI_HIGH_S001 -b tied_unifrac_check -c True -o .
```

## Evaluation signals

- Both computed UniFrac values (Taxy-Pro and MetaPhyler vs. gold standard on CAMI_HIGH_S001) are exactly equal, matching the reported tied score
- unifrac_comparison.csv exists and contains entries for both profilers
- Per-taxon relative abundance differences are recorded and non-empty despite the identical single-value metric
- Submission-to-tool mapping is confirmed via the description.property file in each profiler's subdirectory before computing metrics
- Percentage abundances were derived using the NCBI taxdump database, consistent with TAMPA's preprocessing

## Limitations

- Validation was limited to simulated data; the true ground-truth distribution is only known for simulated data
- Hypotheses generated by TAMPA on real biological data require further analysis and follow-up studies for validation
- Single-value metrics such as the F1 score (and tied UniFrac scores) fail to capture the biological differences between datasets — hence this skill's outputs are differences, not a replacement metric

## Evidence

- [results] Taxy-Pro and MetaPhyler had an identical UniFrac score on a particular sample: "We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler"
- [results] TAMPA computes percentage relative abundance per taxa and identifies which specific clades contributed to metric values: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [methods] Analysis limited to CAMI_HIGH_S001 sample: "We limited our analysis to sample 1 of the high-complexity dataset, denoted by CAMI_HIGH_S001."
- [methods] Profile files and NCBI taxdump used for abundance predictions: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [methods] Profile files extracted from the CAMI challenge GitHub repo: "The profile files were extracted from the GitHub repo of the CAMI challenge: https://github.com/CAMI-challenge/firstchallenge_evaluation/tree/master/profiling/data/profile_submissions"
- [readme] Contrast mode highlights false positives and false negatives: "TAMPA provides a "CONTRAST MODE" to better visualize the differences between the tool and gold standard."
