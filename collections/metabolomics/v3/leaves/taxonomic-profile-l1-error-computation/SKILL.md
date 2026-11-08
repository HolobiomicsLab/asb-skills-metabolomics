---
name: taxonomic-profile-l1-error-computation
description: Use when you need to quantitatively rank taxonomic profilers by L1 norm
  error against a known ground-truth profile for a specific CAMI sample, at a chosen
  taxonomic rank.
when_to_use_negative:
- When no ground-truth taxonomic profile exists (real biological data) — TAMPA is
  then used only for hypothesis generation rather than error-based ranking
- When a single aggregate metric such as UniFrac or F1 is insufficient and you need
  visualization of which specific clades contributed to metric values — use TAMPA's
  contrast/normal visualization modes instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Computes percentage relative abundances per taxon and L1 error between profiler
    output and ground truth; invoked with -l/--plot_l1 flag
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA is integrated;
    alternative for computing profiling metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:7d7b93502e55936b1c0120a4d54da03a3462437bb06a40715f106f265cbdaf0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - OPAL
  - TAMPA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# taxonomic-profile-l1-error-computation

## Summary

Computes per-tool L1 norm error between taxonomic profiler output and a ground-truth CAMI profile using TAMPA, enabling ranking of profilers (e.g. MetaPhyler top, mOTU bottom) on a specific sample such as CAMI_HIGH_S001.

## When to use

Use when you need to quantitatively rank taxonomic profilers by L1 norm error against a known ground-truth profile for a specific CAMI sample, at a chosen taxonomic rank.

## When NOT to use

- When no ground-truth taxonomic profile exists (real biological data) — TAMPA is then used only for hypothesis generation rather than error-based ranking
- When a single aggregate metric such as UniFrac or F1 is insufficient and you need visualization of which specific clades contributed to metric values — use TAMPA's contrast/normal visualization modes instead

## Inputs

- Taxonomic profiler output profile files (e.g. MetaPhyler, mOTU, Taxy-Pro submissions from CAMI first challenge profiling repository)
- Ground-truth (gold standard) taxonomic profile for sample CAMI_HIGH_S001
- NCBI taxdump database
- description.property mapping file from CAMI firstchallenge_evaluation repo (anonymous submission name to tool name)

## Outputs

- Per-profiler L1 error values at the chosen taxonomic rank
- Ranked table of profilers from lowest to highest L1 error (e.g. cami_high_s001_l1_errors.csv)
- Optional L1 error plot from TAMPA

## How to apply

Download profiler submission profiles and the gold-standard ground-truth profile for CAMI_HIGH_S001 from the CAMI first challenge evaluation repository, using description.property to map anonymous submission names to tool names (MetaPhyler, mOTU, Taxy-Pro). Load each profile with the NCBI taxdump database in TAMPA to build percentage relative abundances per taxon. Run TAMPA in L1-error mode with the -l/--plot_l1 flag (e.g. 'python src/tampa.py -i tool.profile -g ground_truth rank -s CAMI_HIGH_S001 -b basename -k linear -r 1600'), optionally adding -n/--normalize so node weights sum to one before error computation. Tabulate L1 error per profiler at the chosen taxonomic rank and sort lowest to highest error to produce the ranking, writing results such as cami_high_s001_l1_errors.csv.

## Related tools

- **TAMPA** (Computes percentage relative abundances per taxon and L1 error between profiler output and ground truth; invoked with -l/--plot_l1 flag) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA is integrated; alternative for computing profiling metrics)

## Examples

```
python src/tampa.py -i tool.profile -g ground_truth.profile rank -s CAMI_HIGH_S001 -b basename -k linear -r 1600 -l -n -o .
```

## Evaluation signals

- Ranking is consistent with the paper's report: MetaPhyler top-performing and mOTU bottom-performing on CAMI_HIGH_S001 under the L1 norm error metric
- L1 errors are non-negative and decrease toward zero as predicted profiles approach the ground truth; normalization (-n) makes node weights sum to one
- Output table (e.g. cami_high_s001_l1_errors.csv) contains one L1 error per profiler evaluated (MetaPhyler, mOTU, Taxy-Pro) at the specified rank
- All profiles were successfully parsed against the NCBI taxdump, producing percentage abundances per taxon without unparsed taxIDs

## Limitations

- Ground truth is only known for simulated data; validation cannot be fully performed on real biological datasets
- Single-value metrics such as the L1 norm may miss biological differences between profiles that TAMPA's visualization exposes
- Rankings are specific to the sample and rank chosen (e.g. CAMI_HIGH_S001); other samples or ranks may yield different orderings

## Evidence

- [results] MetaPhyler top-performing, mOTU bottom-performing under L1: "3 profiles from the publicly available CAMI dataset [1–3]: MetaPhyler [14], mOTU [15], and Taxy-Pro [30]"
- [results] TAMPA computes percentage relative abundance per taxa: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [methods] L1 plotting option in TAMPA: "-l, --plot_l1         If you also want to plot the L1 error"
- [methods] Normalize option in TAMPA: "-n, --normalize       specify this option if you want to normalize the node weights/relative abundances so that they sum to one"
- [methods] NCBI taxdump used to create abundance predictions: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [readme] Example TAMPA CLI invocation: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o ."
- [discussion] Simulated-data-only validation caveat: "Using simulated data was necessary in order to know the true distribution of the ground truth."
- [abstract] F1-style single metrics miss biological differences: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
