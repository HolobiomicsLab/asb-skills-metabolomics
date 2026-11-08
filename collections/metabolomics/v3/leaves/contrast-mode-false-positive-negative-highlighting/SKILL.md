---
name: contrast-mode-false-positive-negative-highlighting
description: Use when comparing one profiler's taxonomic profile against a known ground
  truth (e.g., a CAMI marine sample) and you need to see exactly which taxa are false
  positives/false negatives rather than relying on F1 or L1 scores alone.
when_to_use_negative:
- No ground-truth profile exists — the article notes TAMPA contrast mode requires
  a gold standard to classify false positives/negatives; without one use TAMPA's normal
  mode to compare profiler outputs instead
- You need visualization of multiple profilers' relative abundances rather than profiler-vs-gold-standard
  error analysis (use TAMPA normal mode, c=False)
- Input is amplicon (16S) microbial data — the paper states metacoder-style amplicon
  approaches aside, TAMPA targets WGS metagenomic taxonomic profiles
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxon against the ground truth
    and renders the contrast-mode tree highlighting false positives (red) and false
    negatives (blue)
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Tree-building toolkit used to render the annotated taxonomic tree image
  repo: https://github.com/dkoslicki/TAMPA.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:73c4d5bd95c04f269661bda1a811832744e18300a2a3ec377e296782ff393791
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - TAMPA
  - ete3
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# contrast-mode-false-positive-negative-highlighting

## Summary

TAMPA's contrast mode (c=True) renders a taxonomic tree in which false-positive taxa appear as red circles, false-negative taxa as blue circles, and remaining taxa as a white-to-green gradient proportional to relative error, exposing which clades a profiler misses or hallucinates despite good single-value metrics.

## When to use

Use when comparing one profiler's taxonomic profile against a known ground truth (e.g., a CAMI marine sample) and you need to see exactly which taxa are false positives/false negatives rather than relying on F1 or L1 scores alone.

## When NOT to use

- No ground-truth profile exists — the article notes TAMPA contrast mode requires a gold standard to classify false positives/negatives; without one use TAMPA's normal mode to compare profiler outputs instead
- You need visualization of multiple profilers' relative abundances rather than profiler-vs-gold-standard error analysis (use TAMPA normal mode, c=False)
- Input is amplicon (16S) microbial data — the paper states metacoder-style amplicon approaches aside, TAMPA targets WGS metagenomic taxonomic profiles

## Inputs

- Profiler taxonomic profile file (e.g., data/mad_yalow_0.profile.txt, standard CAMI profile format)
- Ground-truth taxonomic profile file (e.g., data/gs_marine_short.profile.txt)
- NCBI taxdump database (for taxonomy tree construction and percentage abundance predictions)
- Taxonomic rank at which to plot (e.g., 'class' or 'phylum')
- Sample of interest identifier (e.g., marmgCAMI2_short_read_sample_0)

## Outputs

- High-resolution taxonomic tree image (PNG at 1600 dpi) with false positives as red circles, false negatives as blue circles, true positives as white, and remaining taxa on a white-to-green relative-error gradient
- Output file written to the specified output path (-o)

## How to apply

Clone github.com/dkoslicki/TAMPA and activate the 'CAMIViz' conda environment (python=3.7, numpy, ete3, seaborn, pandas, matplotlib, biom-format). Load the profiler profile (e.g., data/mad_yalow_0.profile.txt) and the ground-truth profile (e.g., data/gs_marine_short.profile.txt) together with the NCBI taxdump database to compute percentage abundance predictions per taxon. Run TAMPA in contrast mode at the chosen taxonomic rank ('class' in the paper), activating it with -c True, plus settings like -k linear scaling, -r 1600 dpi, sample -s marmgCAMI2_short_read_sample_0, and -b output base name. The ete3 toolkit tree-building pipeline renders the tree with false-positive taxa as red circles, false-negative taxa as blue circles, true positives as white, and remaining taxa on a white-to-green gradient proportional to relative error. Contrast mode is especially helpful for large trees to identify problematic subregions; use -thr or -top to condense the visualization if needed. On the CAMI marine dataset this mode revealed that MetaPhlAn missed both Tenericutes and Planctomycetes and showed a considerable difference in Firmicutes percentage abundance.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxon against the ground truth and renders the contrast-mode tree highlighting false positives (red) and false negatives (blue)) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Tree-building toolkit used to render the annotated taxonomic tree image) — https://github.com/dkoslicki/TAMPA.git

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o .
```

## Evaluation signals

- The output PNG file exists at the specified output path (-o)
- Visual inspection confirms false-positive taxa are drawn as red circles and false-negative taxa as blue circles per the paper's contrast-mode specification
- Taxa neither false positive nor false negative appear as white or on a white-to-green gradient proportional to relative error
- Highlighted clades match known profiler errors — e.g., on the marine CAMI sample MetaPhlAn misses Tenericutes and Planctomycetes (blue false-negative circles)
- The rendered tree is rooted/organized by NCBI taxdump taxonomy at the requested rank (e.g., 'class')

## Limitations

- Ground truth is only known for simulated data; validation on real biological data is impossible in the same way
- Contrast mode requires a gold-standard profile, so it cannot be applied directly to real datasets lacking ground truth
- Hypotheses generated by TAMPA (e.g., missed taxa) require further analysis and follow-up studies for validation
- For large trees the visualization can be overwhelming; use -thr or -top to condense it

## Evidence

- [methods] a contrast mode, which highlights the false positives and false negatives: "a contrast mode, which highlights the false positives and false negatives"
- [readme] Contrast mode can be used to identify problematic subregions of large trees.: "-c: Contrast mode can be used to identify problematic subregions of large trees."
- [abstract] single metrics, such as the F1 score, fail to capture the biological differences between the datasets: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [methods] python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0"
- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
