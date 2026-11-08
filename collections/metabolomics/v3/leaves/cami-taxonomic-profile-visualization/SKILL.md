---
name: cami-taxonomic-profile-visualization
description: Use when benchmarking taxonomic profilers on CAMI data (e.g. marmgCAMI2
  marine short-read sample) and single-value metrics such as F1 score, L1 norm error,
  or UniFrac fail to show which clades or ranks actually differ.
when_to_use_negative:
- When no ground-truth profile exists — contrast mode (-c True) requires a gold standard
  to define false positives/negatives; use normal mode to compare profiler outputs
  instead
- When the data are amplicon profiles — the paper notes tools like metacoder serve
  amplicon data, while TAMPA targets WGS metagenomic taxonomic profiles
- When the goal is a single numeric benchmark score (F1, L1 norm, UniFrac) rather
  than identifying which clades contributed to the metric differences
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxon from profiler and gold-standard
    profiles and drives the tree plot; -c True enables contrast mode highlighting
    false positives/negatives
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Toolkit used by TAMPA's tree-building pipeline to render the taxonomic tree
    visualization
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA has been integrated
    for benchmarking workflows
  repo: https://github.com/CAMI-challenge/OPAL
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:b067495ae3eac56abf41ed7ffb3c9d673882795358089c9bbf8dd5c2d8ee5199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - OPAL
  - TAMPA
  - ete3
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cami-taxonomic-profile-visualization

## Summary

Visualize and compare taxonomic profiler output against a CAMI gold-standard profile by rendering an ete3 taxonomic tree (TAMPA) whose nodes encode percentage relative abundance, with a contrast mode that marks false positives as red circles and false negatives as blue circles.

## When to use

Use when benchmarking taxonomic profilers on CAMI data (e.g. marmgCAMI2 marine short-read sample) and single-value metrics such as F1 score, L1 norm error, or UniFrac fail to show which clades or ranks actually differ.

## When NOT to use

- When no ground-truth profile exists — contrast mode (-c True) requires a gold standard to define false positives/negatives; use normal mode to compare profiler outputs instead
- When the data are amplicon profiles — the paper notes tools like metacoder serve amplicon data, while TAMPA targets WGS metagenomic taxonomic profiles
- When the goal is a single numeric benchmark score (F1, L1 norm, UniFrac) rather than identifying which clades contributed to the metric differences

## Inputs

- Profiler taxonomic profile file (e.g. data/mad_yalow_0.profile.txt, MetaPhlAn/mOTUs-style profile with @SampleID and TaxID entries)
- Ground-truth taxonomic profile (e.g. data/gs_marine_short.profile.txt for the marmgCAMI2 marine short-read sample)
- NCBI taxdump database (used to create percentage abundance predictions for each taxon)
- Sample identifier of interest (e.g. marmgCAMI2_short_read_sample_0) and target taxonomic rank (e.g. class)

## Outputs

- High-resolution (e.g. 1600 dpi) PNG/SVG/PDF taxonomic tree image rendered with ete3
- Contrast-mode tree with false-positive taxa as red circles and false-negative taxa as blue circles, true positives in white, and remaining taxa on a white-to-green relative-error gradient
- Optionally an L1-error plot (-l) alongside the abundance tree

## How to apply

Clone github.com/dkoslicki/TAMPA and activate the 'CAMIViz' conda environment (python=3.7 with numpy, ete3, seaborn, pandas, matplotlib, biom-format from the defaults, bioconda, conda-forge, and etetoolkit channels). Supply the profiler profile (e.g. data/mad_yalow_0.profile.txt), the ground-truth profile (e.g. data/gs_marine_short.profile.txt), and the NCBI taxdump database, which TAMPA uses to compute percentage abundance predictions per taxon and then relative abundances. Choose the taxonomic rank (e.g. 'class'), the scaling (-k linear), resolution (-r 1600 dpi), and output base name (-b marine_test); set -c True for contrast mode, which highlights false positives (red circles) and false negatives (blue circles) against the gold standard, with remaining taxa on a white-to-green gradient proportional to relative error. These relative abundances feed the ete3 tree-building pipeline that renders the plot; for large trees, use -thr (abundance threshold), -top (top-abundance nodes), or -dt (highlight differences at or above N percent) to condense the visualization. Validate by confirming the output PNG exists at the requested dpi and, by visual inspection, that red/blue circles mark the expected taxa, consistent with the paper's finding that MetaPhlAn missed Tenericutes and Planctomycetes on the marine dataset.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxon from profiler and gold-standard profiles and drives the tree plot; -c True enables contrast mode highlighting false positives/negatives) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Toolkit used by TAMPA's tree-building pipeline to render the taxonomic tree visualization)
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA has been integrated for benchmarking workflows) — https://github.com/CAMI-challenge/OPAL

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o .
```

## Evaluation signals

- Output PNG file exists in the specified output directory at the requested resolution (e.g. -r 1600) for the chosen rank and sample (e.g. class rank, marmgCAMI2_short_read_sample_0, base name marine_test)
- Visual inspection confirms contrast-mode specification: false-positive taxa as red circles, false-negative taxa as blue circles, true positives white, remaining taxa on a white-to-green relative-error gradient
- Run with -c False reproduces the normal-mode abundance tree; toggling to -c True adds the red/blue highlight layer without changing the underlying tree topology
- Tree findings match paper-reported biology: on the marine dataset, MetaPhlAn failed to detect Tenericutes and Planctomycetes and showed considerable abundance differences for groups such as Firmicutes versus mOTUs

## Limitations

- Validation is limited to simulated data — the true ground-truth distribution is only known for simulated (CAMI) data
- TAMPA generates hypotheses on real biological data; further analysis and follow-up studies are required to validate them
- For large trees the full visualization can be unwieldy; use -thr, -top, or contrast mode to isolate problematic subregions, and -dt to highlight taxa differing by at least N percent abundance

## Evidence

- [methods] Contrast mode highlighting false positives and false negatives: "a contrast mode, which highlights the false positives and false negatives"
- [methods] ete3 tree-building pipeline: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [readme] CAMIViz environment creation command: "conda create -c etetoolkit -y -n CAMIViz python=3.7 numpy  ete3  seaborn pandas matplotlib biom-format"
- [abstract] TAMPA hypothesis-generation ability: "the unique ability of TAMPA to generate a novel biological hypothesis by highlighting the taxonomic differences between samples otherwise missed by commonly utilized metrics"
- [results] Marine dataset provenance: "on the marine dataset published in the “Critical Assessment of Metagenome Interpretation: The Second Round of Challenges”"
