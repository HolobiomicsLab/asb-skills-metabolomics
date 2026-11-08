---
name: taxonomic-abundance-difference-extraction
description: Use when two or more taxonomic profilers receive identical single-value
  benchmark scores (e.g., UniFrac, F1) on a CAMI sample, and you need to identify
  the specific clades whose predicted abundances differ.
when_to_use_negative:
- When no ground truth or multiple profiler profiles are available and you only have
  a single profiler's output with nothing to contrast against
- When data are amplicon-based rather than WGS metagenomic taxonomic profiles (metacoder-style
  amplicon tools, not TAMPA-style WGS profiling, is the relevant domain)
- When a single-value metric already fully answers the question and per-clade resolution
  is not needed
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxa, compares profiler outputs,
    and builds ete3 tree visualizations with contrast mode highlighting false positives/false
    negatives
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA is integrated for
    benchmarking profilers
  repo: https://github.com/CAMI-challenge/OPAL
- name: Taxy-Pro
  role: One of the compared CAMI profilers whose tied UniFrac score with MetaPhyler
    motivates abundance-difference extraction
- name: MetaPhyler
  role: The other compared CAMI profiler in the tied-metric case study on CAMI_HIGH_S001
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:f83c77a62fbc83a4b9afdf31e1b1cadfc04ea2f49f6b3d59c9d4d9a1cfa07797
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - MetaPhyler
  - OPAL
  - TAMPA
  - Taxy-Pro
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# taxonomic-abundance-difference-extraction

## Summary

A skill for extracting and visualizing per-taxon relative abundance differences between taxonomic profiler outputs (and against a gold standard), as implemented by TAMPA on CAMI challenge profiling data. It reveals which specific clades drive metric ties or score equivalence that single-value metrics like UniFrac or F1 cannot resolve.

## When to use

Use when two or more taxonomic profilers receive identical single-value benchmark scores (e.g., UniFrac, F1) on a CAMI sample, and you need to identify the specific clades whose predicted abundances differ.

## When NOT to use

- When no ground truth or multiple profiler profiles are available and you only have a single profiler's output with nothing to contrast against
- When data are amplicon-based rather than WGS metagenomic taxonomic profiles (metacoder-style amplicon tools, not TAMPA-style WGS profiling, is the relevant domain)
- When a single-value metric already fully answers the question and per-clade resolution is not needed

## Inputs

- Taxonomic profiler output profile files (e.g., Taxy-Pro, MetaPhyler, mOTU submissions from CAMI profiling/data/profile_submissions)
- Gold-standard (ground truth) taxonomic profile file for the sample (e.g., CAMI_HIGH_S001)
- NCBI taxdump database for mapping taxa to percentage abundance predictions
- description.property files mapping anonymous CAMI submission names to profiler tool names

## Outputs

- Per-taxon relative abundance differences between profiler profiles (and vs. gold standard)
- Taxonomic tree visualization at a chosen rank (e.g., phylum-level tree for CAMI_HIGH_S001)
- Comparison record (e.g., unifrac_comparison.csv) with tied-score verification and abundance differences
- Contrast-mode plot highlighting false positive and false negative taxa

## How to apply

Download the profiler submissions and gold-standard profile files from the CAMI first challenge evaluation repo (profiling/data/profile_submissions), mapping anonymous submission names to tool names via each subdirectory's description.property file. Build percentage abundance predictions for each taxon from the profile files using the NCBI taxdump database, as TAMPA does, then compare percentage abundances across tools to compute per-taxon relative abundance differences. Run TAMPA (ete3-based tree-building pipeline) at a chosen taxonomic rank (e.g., phylum) on the sample of interest such as CAMI_HIGH_S001, optionally using contrast mode (-c True) to highlight false positives and false negatives versus the gold standard. Verify single-value scores first (e.g., confirm tied UniFrac for Taxy-Pro and MetaPhyler) so the abundance-difference extraction explains the tie rather than replacing the benchmark. Record the tied-score verification and per-taxon abundance differences in a comparison record such as unifrac_comparison.csv.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxa, compares profiler outputs, and builds ete3 tree visualizations with contrast mode highlighting false positives/false negatives) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA is integrated for benchmarking profilers) — https://github.com/CAMI-challenge/OPAL
- **Taxy-Pro** (One of the compared CAMI profilers whose tied UniFrac score with MetaPhyler motivates abundance-difference extraction)
- **MetaPhyler** (The other compared CAMI profiler in the tied-metric case study on CAMI_HIGH_S001)

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o .
```

## Evaluation signals

- The computed UniFrac distances of the two profiler profiles against the gold standard are equal (identical to the reported tie for Taxy-Pro and MetaPhyler on CAMI_HIGH_S001)
- The comparison record (unifrac_comparison.csv) exists and contains entries for both profilers
- Per-taxon percentage abundances reconstruct correctly from profile files plus the NCBI taxdump database
- Contrast-mode output flags specific taxa with abundance differences at or above the highlight threshold (-dt) between the two profiles
- Visualized differences (e.g., at the phylum level) are non-empty even though the single-value metric reports no difference

## Limitations

- Validation was limited to simulated data where the true ground-truth distribution is known
- Hypotheses generated by TAMPA on real biological data require further analysis and follow-up studies for validation
- TAMPA supports up to 3 input profilers at once, limiting the number of simultaneously compared tools
- Large trees can be unwieldy; condensation options (-thr, -top) or contrast mode may be needed to identify problematic subregions

## Evidence

- [results] TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [results] We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler: "We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler"
- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [methods] The profile files were extracted from the GitHub repo of the CAMI challenge: https://github.com/CAMI-challenge/firstchallenge_evaluation/tree/master/profiling/data/profile_submissions: "The profile files were extracted from the GitHub repo of the CAMI challenge: https://github.com/CAMI-challenge/firstchallenge_evaluation/tree/master/profiling/data/profile_submissions"
- [abstract] single metrics, such as the F1 score, fail to capture the biological differences between the datasets: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
