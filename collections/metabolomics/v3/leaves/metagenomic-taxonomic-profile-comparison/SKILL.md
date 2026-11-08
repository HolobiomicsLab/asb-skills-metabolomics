---
name: metagenomic-taxonomic-profile-comparison
description: Use when two or more taxonomic profilers report identical or near-identical
  single-value metric scores (e.g., UniFrac) against a gold standard and you must
  determine whether their taxonomic profiles actually differ per taxon.
when_to_use_negative:
- When a gold-standard profile is unavailable — TAMPA's contrast mode requires a ground
  truth input profile and validation relies on knowing the true distribution, which
  is only known for simulated data
- When the goal is amplicon (16S) data analysis — metacoder-style amplicon visualization
  tools are not designed for WGS metagenomic profiles, and this skill is built around
  WGS CAMI profiling data
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxa, visualizes differences between
    profiler profiles and gold standard on a taxonomic tree, and offers contrast mode
    highlighting false positives/negatives
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA is integrated;
    can be used to compute profiling metrics such as UniFrac
  repo: https://github.com/CAMI-challenge/OPAL
- name: Taxy-Pro
  role: One of the two tied profilers whose CAMI_HIGH_S001 profile is compared against
    the gold standard and against MetaPhyler
- name: MetaPhyler
  role: The other tied profiler whose CAMI_HIGH_S001 profile is compared; shares an
    identical UniFrac score with Taxy-Pro despite per-taxon differences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:e7f946e203955863403d70943f9ccbfb7a6d01f24e73a3cf2c97cfb1a65d9857
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - MetaPhyler
  - OPAL
  - TAMPA
  - Taxy-Pro
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metagenomic-taxonomic-profile-comparison

## Summary

Reproduce tied single-value metric cases (e.g., identical UniFrac scores) by comparing taxonomic profiler outputs (Taxy-Pro vs MetaPhyler on CAMI_HIGH_S001) at the per-taxon relative abundance level using TAMPA, revealing biological differences that single-value metrics miss.

## When to use

Use when two or more taxonomic profilers report identical or near-identical single-value metric scores (e.g., UniFrac) against a gold standard and you must determine whether their taxonomic profiles actually differ per taxon.

## When NOT to use

- When a gold-standard profile is unavailable — TAMPA's contrast mode requires a ground truth input profile and validation relies on knowing the true distribution, which is only known for simulated data
- When the goal is amplicon (16S) data analysis — metacoder-style amplicon visualization tools are not designed for WGS metagenomic profiles, and this skill is built around WGS CAMI profiling data

## Inputs

- Taxy-Pro taxonomic profile file for CAMI_HIGH_S001 (from CAMI firstchallenge_evaluation profile_submissions)
- MetaPhyler taxonomic profile file for CAMI_HIGH_S001 (from CAMI firstchallenge_evaluation profile_submissions)
- Gold-standard (ground truth) taxonomic profile for CAMI_HIGH_S001
- description.property files mapping anonymous submission names to tool names
- NCBI taxdump database

## Outputs

- Computed UniFrac distances of Taxy-Pro and MetaPhyler profiles vs the gold standard (expected identical)
- Per-taxon relative abundance differences between the two profiler profiles (e.g., at the phylum level)
- Comparison record file (unifrac_comparison.csv) with tie verification and abundance differences
- Optional TAMPA tree visualization (e.g., metaphyler_CAMIhigh_tree_phylum_CAMI_HIGH_S001.png)

## How to apply

Download the profiler submissions (Taxy-Pro, MetaPhyler) and the gold-standard profile for CAMI_HIGH_S001 from the CAMI first challenge evaluation repo's profiling/data/profile_submissions directory, and map anonymous submission IDs to tool names via each directory's description.property file. Using the NCBI taxdump database, build percentage abundance predictions for each taxon from the profile files as TAMPA does, then compute the UniFrac distance of both profiler profiles against the gold standard for the sample. Verify the two UniFrac values are identical (the reported tie), then use TAMPA's per-taxon percentage relative abundance comparison — optionally with contrast mode (-c True) to highlight false positives and false negatives — to record the specific per-taxon abundance differences between the two profiles. Write the tie verification and abundance differences into a comparison record (e.g., unifrac_comparison.csv); the record is valid if both computed UniFrac values are equal and entries exist for both profilers.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxa, visualizes differences between profiler profiles and gold standard on a taxonomic tree, and offers contrast mode highlighting false positives/negatives) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA is integrated; can be used to compute profiling metrics such as UniFrac) — https://github.com/CAMI-challenge/OPAL
- **Taxy-Pro** (One of the two tied profilers whose CAMI_HIGH_S001 profile is compared against the gold standard and against MetaPhyler)
- **MetaPhyler** (The other tied profiler whose CAMI_HIGH_S001 profile is compared; shares an identical UniFrac score with Taxy-Pro despite per-taxon differences)

## Examples

```
python src/tampa.py -i metaphyler.profile.txt -g gs_camihigh_s001.profile.txt class -s CAMI_HIGH_S001 -c True -nm -o .
```

## Evaluation signals

- The computed UniFrac distance of the Taxy-Pro profile against the gold standard equals the UniFrac distance of the MetaPhyler profile against the gold standard for CAMI_HIGH_S001
- Both computed UniFrac values match the reported tied score from the CAMI first challenge evaluation
- unifrac_comparison.csv exists and contains entries for both profilers (Taxy-Pro and MetaPhyler)
- Per-taxon relative abundance differences are non-empty and reproducible at specific ranks (e.g., phylum-level differences as in Fig. 1), even though the single-value metric reports no difference
- Percentage abundance predictions derived from profile files using the NCBI taxdump database reproduce TAMPA's per-taxon abundances

## Limitations

- Validation was limited to simulated data; the true ground-truth distribution is only known for simulated data
- Hypotheses generated by TAMPA on real biological data require further analysis and follow-up studies for validation
- Single-value metrics such as the F1 score (and UniFrac ties) fail to capture biological differences between datasets, so metric equality does not imply profile equality

## Evidence

- [results] Taxy-Pro and MetaPhyler had an identical UniFrac score on a particular sample: "We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler"
- [methods] Profile files extracted from CAMI challenge repo: "The profile files were extracted from the GitHub repo of the CAMI challenge: https://github.com/CAMI-challenge/firstchallenge_evaluation/tree/master/profiling/data/profile_submissions"
- [methods] Percentage abundances from taxdump: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [methods] Sample limited to CAMI_HIGH_S001: "We limited our analysis to sample 1 of the high-complexity dataset, denoted by CAMI_HIGH_S001."
- [readme] Contrast mode in README: "TAMPA provides a "CONTRAST MODE" to better visualize the differences between the tool and gold standard."
