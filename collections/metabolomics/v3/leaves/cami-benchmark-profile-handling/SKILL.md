---
name: cami-benchmark-profile-handling
description: Use when input data are CAMI-format taxonomic profiles (e.g., from CAMI-challenge/firstchallenge_evaluation
  profile_submissions) and the goal is to reproduce profiler benchmarking, tie-breaking,
  or per-taxon abundance comparisons against a gold standard.
when_to_use_negative:
- Input data are amplicon (16S) datasets — metacoder is the amplicon-oriented alternative
  and TAMPA/this skill targets WGS metagenomic taxonomic profiles
- No gold-standard or benchmark context exists and the goal is to choose a profiler
  on real data — TAMPA can still be used for hypothesis generation, but the benchmark-tie/L1-reproduction
  workflows here require a ground-truth profile
- Input is raw sequencing data rather than an already-computed taxonomic profile;
  this skill starts from standardized profiler output
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: 'Core analysis/visualization engine: computes percentage relative abundance
    per taxon, builds ete3 trees, and offers normal, contrast, and L1-error modes
    over CAMI profiles'
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Benchmarking platform into which TAMPA is integrated for taxonomic profiling
    evaluation; also used to compute metrics such as UniFrac against gold standards
  repo: https://github.com/CAMI-challenge/firstchallenge_evaluation
- name: ete3
  role: Toolkit used by the tree-building pipeline to attach taxa via the NCBI taxonomy
    interface and render the annotated taxonomic tree
- name: MetaPhyler
  role: CAMI profiler whose submissions are downloaded, name-mapped, and benchmarked
    (top-performing under L1 norm on CAMI_HIGH_S001)
- name: mOTU
  role: CAMI profiler benchmarked in the comparison workflows (bottom-performing under
    L1 norm on CAMI_HIGH_S001)
- name: Taxy-Pro
  role: CAMI profiler benchmarked in the tied-UniFrac case study against MetaPhyler
    on CAMI_HIGH_S001
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:df53466003aadb7d139b88456c4d1483a548beebf8de00201c649227faf0dcb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - MetaPhyler
  - OPAL
  - TAMPA
  - Taxy-Pro
  - ete3
  - mOTU
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cami-benchmark-profile-handling

## Summary

Handles taxonomic profiler output from the CAMI challenge evaluation: downloads submitted and gold-standard CAMI-format profile files, maps anonymous submission names to tool names, and builds per-taxon percentage abundance predictions for comparison with TAMPA and metrics such as UniFrac and the L1 norm.

## When to use

Use when input data are CAMI-format taxonomic profiles (e.g., from CAMI-challenge/firstchallenge_evaluation profile_submissions) and the goal is to reproduce profiler benchmarking, tie-breaking, or per-taxon abundance comparisons against a gold standard.

## When NOT to use

- Input data are amplicon (16S) datasets — metacoder is the amplicon-oriented alternative and TAMPA/this skill targets WGS metagenomic taxonomic profiles
- No gold-standard or benchmark context exists and the goal is to choose a profiler on real data — TAMPA can still be used for hypothesis generation, but the benchmark-tie/L1-reproduction workflows here require a ground-truth profile
- Input is raw sequencing data rather than an already-computed taxonomic profile; this skill starts from standardized profiler output

## Inputs

- CAMI-format taxonomic profile files from profiler submissions (profiling/data/profile_submissions in CAMI-challenge/firstchallenge_evaluation)
- Gold-standard (ground truth) CAMI profile files (e.g., gs_marine_short.profile.txt)
- description.property files mapping anonymous submission names to tool names
- NCBI taxdump database

## Outputs

- Per-taxon percentage abundance predictions attached to NCBI taxonomy nodes
- Metric comparison records (e.g., unifrac_comparison.csv, cami_high_s001_l1_errors.csv)
- TAMPA tree/plot outputs (e.g., PNG at user-specified dpi, Newick tree files)
- Profiler rankings under single-value metrics (UniFrac, L1 norm, F1 score)

## How to apply

Download profiler submissions and the gold-standard profile for the target sample (e.g., CAMI_HIGH_S001 from the high-complexity dataset, or marmgCAMI2_short_read_sample_0 from the CAMI second-round marine dataset) from the CAMI first challenge evaluation GitHub repo under profiling/data/profile_submissions. Use each profiler subdirectory's description.property file to map anonymous submission names to tool names (e.g., Taxy-Pro, MetaPhyler, mOTU). Combine the profile files with the NCBI taxdump database to create percentage abundance predictions for each taxon, then feed them to TAMPA (normal mode, contrast mode, L1-error mode via -l, normalization via -n). Judge correctness by confirming metric tie/rank reproduction (e.g., identical UniFrac for Taxy-Pro and MetaPhyler; MetaPhyler top and mOTU bottom under L1 norm) and by validating that output files exist, parse (e.g., valid Newick trees), and contain taxIDs from the input profiles.

## Related tools

- **TAMPA** (Core analysis/visualization engine: computes percentage relative abundance per taxon, builds ete3 trees, and offers normal, contrast, and L1-error modes over CAMI profiles) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Benchmarking platform into which TAMPA is integrated for taxonomic profiling evaluation; also used to compute metrics such as UniFrac against gold standards) — https://github.com/CAMI-challenge/firstchallenge_evaluation
- **ete3** (Toolkit used by the tree-building pipeline to attach taxa via the NCBI taxonomy interface and render the annotated taxonomic tree)
- **MetaPhyler** (CAMI profiler whose submissions are downloaded, name-mapped, and benchmarked (top-performing under L1 norm on CAMI_HIGH_S001))
- **mOTU** (CAMI profiler benchmarked in the comparison workflows (bottom-performing under L1 norm on CAMI_HIGH_S001))
- **Taxy-Pro** (CAMI profiler benchmarked in the tied-UniFrac case study against MetaPhyler on CAMI_HIGH_S001)

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o .
```

## Evaluation signals

- Computed UniFrac values for Taxy-Pro and MetaPhyler on CAMI_HIGH_S001 are identical, matching the reported tied score, and the unifrac_comparison.csv record contains entries for both profilers
- L1-norm ranking on CAMI_HIGH_S001 places MetaPhyler top-performing and mOTU bottom-performing, consistent with the paper's report
- Output files exist and parse correctly (e.g., Newick trees contain nodes for taxIDs present in the input profile; PNG saved at the specified 1600 dpi)
- Every anonymous submission name has been resolved to a tool name via its description.property file before ranking
- Per-taxon relative abundances differ between the tied-UniFrac profilers (confirming single-value metrics miss biological differences)

## Limitations

- Validation was limited to simulated data; the true ground-truth distribution is only known for simulated data
- Hypotheses generated by TAMPA on real biological data require further analysis and follow-up studies for validation
- Single-value metrics such as the F1 score (and tied UniFrac scores) fail to capture biological differences between profiles, so metric agreement does not imply profile agreement

## Evidence

- [methods] The profile files were extracted from the GitHub repo of the CAMI challenge: https://github.com/CAMI-challenge/firstchallenge_evaluation/tree/master/profiling/data/profile_submissions: "The profile files were extracted from the GitHub repo of the CAMI challenge: https://github.com/CAMI-challenge/firstchallenge_evaluation/tree/master/profiling/data/profile_submissions"
- [results] We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler: "We choose 2 profilers with an identical UniFrac score on a particular sample [28], Taxy-Pro and Metaphyler"
- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [abstract] single metrics, such as the F1 score, fail to capture the biological differences between the datasets: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [discussion] Using simulated data was necessary in order to know the true distribution of the ground truth.: "Using simulated data was necessary in order to know the true distribution of the ground truth."
