---
name: unifrac-phylogenetic-distance-computation
description: Use when two taxonomic profilers report identical single-value metrics
  (e.g., UniFrac) against a gold standard and you need to verify the tie and identify
  which clades actually differ in relative abundance.
when_to_use_negative:
- When the profiles of interest already have distinct single-value scores — a tie
  is the trigger for this deep comparison
- 'When analyzing amplicon data: TAMPA is designed for WGS metagenomic taxonomic profiles,
  and tools like metacoder are not suitable for WGS profiles'
- When no gold-standard/ground-truth profile exists — UniFrac against ground truth
  requires simulated data where the true distribution is known
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxon and visualizes clade-level
    differences between tied profilers and the gold standard
  repo: https://github.com/dkoslicki/TAMPA
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA is integrated for
    metric computation
- name: Taxy-Pro
  role: One of the two tied profilers whose CAMI_HIGH_S001 profile is compared against
    the gold standard
- name: MetaPhyler
  role: The other tied profiler whose CAMI_HIGH_S001 profile is compared against the
    gold standard
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:23e0e250ec3439f6c9ab20f5001903f518db25dc264cb1847bbd4f24fb867898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA
  tools:
  - MetaPhyler
  - OPAL
  - TAMPA
  - Taxy-Pro
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# unifrac-phylogenetic-distance-computation

## Summary

Compute and verify tied UniFrac phylogenetic distances between taxonomic profiler outputs and a gold-standard profile (e.g., Taxy-Pro vs MetaPhyler on CAMI_HIGH_S001), then use TAMPA to reveal per-taxon abundance differences that the identical single-value score conceals.

## When to use

Use when two taxonomic profilers report identical single-value metrics (e.g., UniFrac) against a gold standard and you need to verify the tie and identify which clades actually differ in relative abundance.

## When NOT to use

- When the profiles of interest already have distinct single-value scores — a tie is the trigger for this deep comparison
- When analyzing amplicon data: TAMPA is designed for WGS metagenomic taxonomic profiles, and tools like metacoder are not suitable for WGS profiles
- When no gold-standard/ground-truth profile exists — UniFrac against ground truth requires simulated data where the true distribution is known

## Inputs

- Taxy-Pro taxonomic profile file for CAMI_HIGH_S001 (from CAMI firstchallenge_evaluation profile_submissions)
- MetaPhyler taxonomic profile file for CAMI_HIGH_S001
- Gold-standard (ground truth) taxonomic profile for CAMI_HIGH_S001
- description.property files mapping anonymous submission IDs to tool names
- NCBI taxdump database

## Outputs

- Computed UniFrac distance values for each profiler versus the gold standard on CAMI_HIGH_S001
- Per-taxon relative abundance differences between the two tied profilers
- unifrac_comparison.csv containing the tied-score verification and abundance difference entries for both profilers

## How to apply

Download the profiler submissions and gold-standard profiles from the CAMI firstchallenge_evaluation repo (profiling/data/profile_submissions) and map anonymous submission names to tool names via each subdirectory's description.property file. Build percentage abundance predictions per taxon from the profile files using the NCBI taxdump database, as TAMPA does. Compute the UniFrac distance of each profiler's profile (e.g., Taxy-Pro and MetaPhyler on CAMI_HIGH_S001) against the gold standard and confirm the two values are identical, reproducing the reported tie. Record the per-taxon relative abundance differences between the two profiles (e.g., at the phylum level, as in TAMPA Fig. 1) and write the verification plus abundance differences into a comparison record such as unifrac_comparison.csv.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxon and visualizes clade-level differences between tied profilers and the gold standard) — https://github.com/dkoslicki/TAMPA
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA is integrated for metric computation)
- **Taxy-Pro** (One of the two tied profilers whose CAMI_HIGH_S001 profile is compared against the gold standard)
- **MetaPhyler** (The other tied profiler whose CAMI_HIGH_S001 profile is compared against the gold standard)

## Examples

```
python src/tampa.py -i data/metaphyler.profile.txt -g data/gs_camihigh.profile.txt class -s CAMI_HIGH_S001 -b camihigh_unifrac_tie -k linear -r 1600 -c True -o .
```

## Evaluation signals

- The UniFrac values computed for Taxy-Pro and MetaPhyler against the gold standard on CAMI_HIGH_S001 are exactly equal (identical to the reported tie)
- unifrac_comparison.csv exists and contains entries for both profilers
- Per-taxon relative abundance differences computed from the profile files and NCBI taxdump reveal specific clade differences (e.g., at the phylum level) despite the tied single-value score
- Submission names are correctly mapped to Taxy-Pro and MetaPhyler via the description.property files before comparison

## Limitations

- Single-value metrics such as UniFrac and F1 score cannot capture biological differences between profiles — tied scores mask real clade-level discrepancies
- Validation against ground truth is only possible for simulated data where the true distribution is known (e.g., CAMI_HIGH_S001)
- Hypotheses generated by TAMPA on real biological data require further follow-up studies for validation

## Evidence

- [results] We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler: "We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler"
- [abstract] Despite sharing an identical UniFrac score, Taxy-Pro and MetaPhyler produce taxonomic profiles with specific differences: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [results] TAMPA computes percentage relative abundance per taxa and identifies which specific clades contributed to metric values: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [methods] Profile files extracted from the CAMI challenge GitHub repo: "The profile files were extracted from the GitHub repo of the CAMI challenge: https://github.com/CAMI-challenge/firstchallenge_evaluation/tree/master/profiling/data/profile_submissions"
- [readme] README example invocation with sample of interest and gold standard: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o ."
