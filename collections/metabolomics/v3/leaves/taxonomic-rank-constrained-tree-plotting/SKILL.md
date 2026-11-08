---
name: taxonomic-rank-constrained-tree-plotting
description: Use when you have one or more taxonomic profiler profiles plus a ground-truth
  CAMI-style profile and need to visualize which clades at a chosen rank (e.g., class,
  phylum) drive profiler errors or abundance differences.
when_to_use_negative:
- Input is amplicon (16S) microbial data rather than WGS metagenomic taxonomic profiles
  — metacoder-style tools aside, TAMPA is designed for WGS profiles
- You need a single summary metric (e.g., F1 score, L1 norm, UniFrac) rather than
  a clade-level visualization of where profilers disagree
- No ground-truth profile exists for contrast mode, which specifically requires comparing
  a tool profile against a gold standard
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxon, compares profiler output
    against ground truth, and drives the rank-constrained tree rendering in normal
    or contrast mode
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Toolkit used by TAMPA's tree-building pipeline to render the taxonomic tree
    plot with the desired node coloring and labels
- name: OPAL
  role: Benchmarking platform into which TAMPA has been integrated for taxonomic profiler
    assessment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:b0f283a4e1a59a760d8d166feb748a08951eb83d1f0019100aa34a7bb6d5c518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - OPAL
  - TAMPA
  - ete3
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# taxonomic-rank-constrained-tree-plotting

## Summary

Render taxonomic profiler output as a rank-constrained (e.g., phylum/class) tree using TAMPA's ete3-based pipeline, optionally in contrast mode where false-positive taxa appear as red circles and false-negative taxa as blue circles, revealing biological differences that single-valued metrics like F1 score miss.

## When to use

Use when you have one or more taxonomic profiler profiles plus a ground-truth CAMI-style profile and need to visualize which clades at a chosen rank (e.g., class, phylum) drive profiler errors or abundance differences.

## When NOT to use

- Input is amplicon (16S) microbial data rather than WGS metagenomic taxonomic profiles — metacoder-style tools aside, TAMPA is designed for WGS profiles
- You need a single summary metric (e.g., F1 score, L1 norm, UniFrac) rather than a clade-level visualization of where profilers disagree
- No ground-truth profile exists for contrast mode, which specifically requires comparing a tool profile against a gold standard

## Inputs

- Profiler taxonomic profile file (e.g., data/mad_yalow_0.profile.txt)
- Ground-truth taxonomic profile file (e.g., data/gs_marine_short.profile.txt)
- NCBI taxdump database
- Taxonomic rank argument (e.g., 'class', phylum)
- Sample identifier (e.g., marmgCAMI2_short_read_sample_0)

## Outputs

- Rendered taxonomic tree image (PNG/SVG/PDF) in the output directory, with false-positive taxa as red circles, false-negative taxa as blue circles, true positives as white, and remaining taxa on a white-to-green relative-error gradient (contrast mode)

## How to apply

Clone github.com/dkoslicki/TAMPA and activate the 'CAMIViz' conda environment (python=3.7, numpy, ete3, seaborn, pandas, matplotlib, biom-format). Run src/tampa.py with the profiler profile (-i), ground-truth profile (-g), a taxonomic rank as the positional argument (e.g., 'class' or phylum), sample name (-s marmgCAMI2_short_read_sample_0), output base name (-b), scaling (-k linear), and resolution (-r 1600). Pass -c True to activate contrast mode, which highlights false positives and false negatives rather than showing the normal white-to-green relative-error gradient. Percentage abundance predictions are computed from the profile files and the NCBI taxdump database; relative abundances feed the ete3 tree-building pipeline that renders and saves the plot (optionally with -nm on headless servers to save directly to file).

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxon, compares profiler output against ground truth, and drives the rank-constrained tree rendering in normal or contrast mode) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Toolkit used by TAMPA's tree-building pipeline to render the taxonomic tree plot with the desired node coloring and labels)
- **OPAL** (Benchmarking platform into which TAMPA has been integrated for taxonomic profiler assessment)

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o .
```

## Evaluation signals

- Output PNG file exists at the specified output path with the expected base name (e.g., marine_test tree at 1600 dpi)
- Visual inspection confirms false-positive taxa are drawn as red circles and false-negative taxa as blue circles per the contrast-mode specification
- Tree nodes are restricted to the requested taxonomic rank (e.g., 'class') rather than mixing ranks
- In normal mode, remaining taxa display a white-to-green gradient proportional to relative error between profiler and ground truth
- Detected missed taxa match known findings, e.g., MetaPhlAn failing to detect Tenericutes and Planctomycetes on the marine CAMI sample

## Limitations

- Validation was limited to simulated data; the true ground-truth distribution is only known for simulated data
- Hypotheses generated by TAMPA (e.g., missed phyla) are hypothesis-generation on real data and require follow-up studies for validation
- The CAMIViz environment requires python=3.7 with ete3 from the etetoolkit channel; Bioconda installation was still awaiting a merged pull request

## Evidence

- [methods] a contrast mode, which highlights the false positives and false negatives: "a contrast mode, which highlights the false positives and false negatives"
- [methods] ete3 tree-building pipeline: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [methods] profile files and NCBI taxdump create abundance predictions: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [abstract] TAMPA single metrics fail to capture biological differences: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [readme] README contrast mode command with -c True: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o ."
- [readme] README large-dataset condensing options: "1. -thr: This option can be used to display only the nodes with abundance higher than a particular threshold"
