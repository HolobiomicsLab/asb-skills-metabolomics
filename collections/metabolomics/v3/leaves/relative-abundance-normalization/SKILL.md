---
name: relative-abundance-normalization
description: Use when comparing raw taxonomic profiler profile files (e.g., CAMI profile_submissions
  and gold-standard profiles) whose relative abundances must be recomputed against
  the NCBI taxdump before ranking tools by error metrics like the L1 norm.
when_to_use_negative:
- Data is amplicon-based rather than WGS metagenomic — the article notes metacoder
  'is not designed for WGS metagenomic analyses', and TAMPA targets WGS profiler outputs
- Profiles are already normalized percentages summing to one, so the -n flag would
  be redundant or distort comparison
- Only a single profiler's output exists with no ground-truth or second profile to
  compare abundances against
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3307
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxa from profile files and the
    NCBI taxdump; its -n/--normalize flag ensures node weights sum to one before metric
    computation
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA has been integrated
    for normalized profiler comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:5b329402093b1a44d14771e5d99be5b3d9cb38c32e7b345555e90bbd2d5a83a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - OPAL
  - TAMPA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# relative-abundance-normalization

## Summary

Normalizes taxonomic profiler outputs into percentage relative abundances per taxon using TAMPA, so that node weights across MetaPhyler, mOTU, Taxy-Pro and ground-truth CAMI profiles are comparable and sum to one before metric computation (e.g., L1 error).

## When to use

Use when comparing raw taxonomic profiler profile files (e.g., CAMI profile_submissions and gold-standard profiles) whose relative abundances must be recomputed against the NCBI taxdump before ranking tools by error metrics like the L1 norm.

## When NOT to use

- Data is amplicon-based rather than WGS metagenomic — the article notes metacoder 'is not designed for WGS metagenomic analyses', and TAMPA targets WGS profiler outputs
- Profiles are already normalized percentages summing to one, so the -n flag would be redundant or distort comparison
- Only a single profiler's output exists with no ground-truth or second profile to compare abundances against

## Inputs

- Taxonomic profiler output profile files (e.g., from the CAMI first challenge evaluation profiling/data/profile_submissions)
- Ground-truth taxonomic profile (gold standard) for the sample of interest
- NCBI taxdump database files
- description.property file mapping anonymous CAMI submission names to tool names (MetaPhyler, mOTU, Taxy-Pro)

## Outputs

- Percentage relative abundance per taxon (node weights) per profiler at the chosen rank
- Normalized relative abundances summing to one, suitable for downstream metric computation (e.g., L1 error)
- Visualization tree plots (png/svg/pdf via -t) of tool vs. ground-truth abundance

## How to apply

Load each profiler's profile file together with the NCBI taxdump database in TAMPA to build percentage relative abundance predictions for each taxon; TAMPA then compares the percentage abundances of each tool to compute the relative abundances. Run TAMPA with the -n/--normalize option so that node weights/relative abundances sum to one before error computation. Work at a chosen taxonomic rank (positional argument, e.g., 'class') and optionally set the -k scaling (e.g., 'linear') and -r resolution flags. On the high-complexity CAMI sample CAMI_HIGH_S001, this normalization step feeds the L1-error ranking in which MetaPhyler is top-performing and mOTU bottom-performing. Optionally apply -top or -thr to condense large trees to the highest-abundance nodes for visualization.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxa from profile files and the NCBI taxdump; its -n/--normalize flag ensures node weights sum to one before metric computation) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA has been integrated for normalized profiler comparison)

## Examples

```
python src/tampa.py -i tool.profile -g ground_truth.profile class -s CAMI_HIGH_S001 -b basename -k linear -r 1600 -n
```

## Evaluation signals

- After applying -n/--normalize, node weights/relative abundances for each profiler sum to one within numerical tolerance
- Percentage abundances are consistent with the NCBI taxdump hierarchy (child taxa aggregate correctly at the chosen rank, e.g., class)
- Downstream L1-error ranking on CAMI_HIGH_S001 reproduces the paper's result: MetaPhyler lowest (top-performing) and mOTU highest (bottom-performing) error
- Normalized profiles support tree plots whose taxon labels match per-taxon percentage abundances in the source profile files

## Limitations

- Validation was limited to simulated data; the true ground-truth distribution is only known for simulated data
- Large taxonomic trees may need condensing via -thr, -top, or contrast mode; unnormalized raw profiler weights can render comparisons misleading

## Evidence

- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [results] TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [readme] -n, --normalize specify this option if you want to normalize the node weights/relative abundances so that they sum to one: "-n, --normalize       specify this option if you want to normalize the node
                        weights/relative abundances so that they sum to one"
- [methods] TAMPA then compares the percentage abundances of each tool to compute the relative abundances.: "TAMPA then compares the percentage abundances of each tool to compute the relative abundances."
