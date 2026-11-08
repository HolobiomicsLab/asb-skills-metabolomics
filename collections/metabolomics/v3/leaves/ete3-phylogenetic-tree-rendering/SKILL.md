---
name: ete3-phylogenetic-tree-rendering
description: Use when comparing CAMI-format taxonomic profiler outputs (e.g. MetaPhlAn,
  mOTUs, MetaPhyler, Taxy-Pro) against a ground-truth profile and you need a phylogenetic
  tree plot showing where abundance errors or missed clades occur.
when_to_use_negative:
- Input is amplicon microbial data — metacoder-style tools exist, but TAMPA targets
  WGS metagenomic taxonomic profiles, and amplicon visualizers are unsuitable for
  WGS profiles
- You only need a single aggregate metric — the paper's point is that single metrics
  like F1 score or L1 norm error fail to capture biological differences, so tree rendering
  is for interpreting differences, not summarizing them
- No taxonomic rank/taxdump mapping is available for the profile taxIDs, since the
  tree is built on the NCBI taxonomy
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Computes per-taxon percentage relative abundances from the profile vs. ground
    truth and drives the tree-building/visualization pipeline
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Toolkit used to construct and render the taxonomy tree with the computed relative
    abundances placed on nodes
- name: OPAL
  role: Benchmarking platform into which TAMPA's tree visualization has been integrated
  repo: https://github.com/CAMI-challenge/OPAL
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:98eca5a09da8c8868f4de5535152f52beafc101ca5f8ab478115d6924bbd1a87
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
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ete3-phylogenetic-tree-rendering

## Summary

Render a taxonomic profile onto an NCBI taxonomy tree using the ete3 toolkit within TAMPA, so that per-taxon percentage relative abundances and (in contrast mode) false-positive/false-negative taxa are visualized on the tree.

## When to use

Use when comparing CAMI-format taxonomic profiler outputs (e.g. MetaPhlAn, mOTUs, MetaPhyler, Taxy-Pro) against a ground-truth profile and you need a phylogenetic tree plot showing where abundance errors or missed clades occur.

## When NOT to use

- Input is amplicon microbial data — metacoder-style tools exist, but TAMPA targets WGS metagenomic taxonomic profiles, and amplicon visualizers are unsuitable for WGS profiles
- You only need a single aggregate metric — the paper's point is that single metrics like F1 score or L1 norm error fail to capture biological differences, so tree rendering is for interpreting differences, not summarizing them
- No taxonomic rank/taxdump mapping is available for the profile taxIDs, since the tree is built on the NCBI taxonomy

## Inputs

- CAMI-format taxonomic profile file(s) (e.g. data/mad_yalow_0.profile.txt)
- Ground-truth taxonomic profile (e.g. data/gs_marine_short.profile.txt)
- NCBI taxdump database
- Sample identifier (e.g. marmgCAMI2_short_read_sample_0)
- Taxonomic rank at which to plot (e.g. 'class', 'phylum')

## Outputs

- Rendered tree image (PNG/SVG/PDF per -t) saved to the output path/figures directory
- Annotated ete3 tree structure of relative abundances per taxon (optionally Newick-format for downstream comparison)

## How to apply

Parse the CAMI-format profile file(s) and, together with the NCBI taxdump database, compute percentage abundance predictions for each taxon; TAMPA compares the percentage abundances to compute relative abundances, which feed the tree-building pipeline. Set up a conda environment (CAMIViz, python=3.7 with numpy, ete3, seaborn, pandas, matplotlib, biom-format), then invoke tampa.py with the taxonomic rank (e.g. 'class'), the input and ground-truth profiles, a scaling key such as '-k linear', resolution (e.g. -r 1600), and an output base name/sample. Enable contrast mode with '-c True' to highlight false positives (red circles), false negatives (blue circles), true positives (white), and remaining taxa on a white-to-green gradient proportional to relative error. For large trees, condense the visualization using -thr (abundance threshold) or -top (top abundant nodes). Save the rendered image to the figures directory and verify it by checking the PNG exists and visually inspecting the color-coded nodes.

## Related tools

- **TAMPA** (Computes per-taxon percentage relative abundances from the profile vs. ground truth and drives the tree-building/visualization pipeline) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Toolkit used to construct and render the taxonomy tree with the computed relative abundances placed on nodes)
- **OPAL** (Benchmarking platform into which TAMPA's tree visualization has been integrated) — https://github.com/CAMI-challenge/OPAL

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o .
```

## Evaluation signals

- The output PNG file exists in the specified output/figures directory
- Visual inspection confirms false-positive taxa appear as red circles and false-negative taxa as blue circles in contrast mode
- The emitted tree parses as valid Newick and contains nodes corresponding to the taxIDs present in the input profile
- Scaling and resolution parameters (e.g. -k linear, -r 1600 dpi) are honored in the rendered figure
- For contrast mode on the marine CAMI sample, known misses (e.g. Tenericutes, Planctomycetes) should appear as highlighted nodes

## Limitations

- Validation is on simulated data where the true ground-truth distribution is known; on real biological data TAMPA is for hypothesis generation requiring follow-up studies
- Environment pins python=3.7 (CAMIViz conda env), which may conflict with modern Python stacks
- Bioconda installation was 'waiting for pull request to get merged' per the README, so conda-from-source is the reliable install path

## Evidence

- [methods] These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot.: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [readme] In the contrast mode, the false positive taxas are represented as: "In the contrast mode, the false positive taxas are represented as"
- [results] TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [readme] -thr: This option can be used to display only the nodes with abundance higher than a particular threshold: "-thr: This option can be used to display only the nodes with abundance higher than a particular threshold"
