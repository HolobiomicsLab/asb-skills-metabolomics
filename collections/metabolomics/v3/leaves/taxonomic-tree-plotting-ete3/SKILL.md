---
name: taxonomic-tree-plotting-ete3
description: Use when standardized taxonomic profiles (profiler output plus a gold-standard
  profile with TaxIDs) need a rank-resolved tree visualization of per-clade relative
  abundances, e.g., MetaPhlAn vs mOTUs on the CAMI marmgCAMI2 marine sample.
when_to_use_negative:
- A single-value metric (e.g., F1 score or UniFrac) already answers the comparison
  question and no clade-level biological interpretation is needed
- Input is raw sequencing reads or binning output rather than a standardized taxonomic
  profile with TaxIDs that can be compared to a ground-truth profile
- A definitive, validated quantitative result on real (non-simulated) biological data
  is required, since TAMPA output is hypothesis-generating
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3293
tools:
- name: TAMPA
  role: Python implementation that computes percentage relative abundance per taxon
    and drives the ete3 tree-building pipeline in normal and contrast modes
  repo: https://github.com/dkoslicki/TAMPA
- name: ete3
  role: Tree-building toolkit invoked by TAMPA to render the abundance-annotated taxonomic
    tree plot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:e9b681d8b45f2886df09dd5e1f345b30dfe21b530a9044fd50279ab919e15fcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA
  tools:
  - TAMPA
  - ete3
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# taxonomic-tree-plotting-ete3

## Summary

TAMPA renders taxonomic abundance profiles as abundance-annotated taxonomic trees using the ete3 toolkit, comparing up to 3 profilers against a gold-standard profile (e.g., MetaPhlAn vs mOTUs on the CAMI marmgCAMI2 marine sample) to expose per-clade differences that single-value metrics like the F1 score miss.

## When to use

Use when standardized taxonomic profiles (profiler output plus a gold-standard profile with TaxIDs) need a rank-resolved tree visualization of per-clade relative abundances, e.g., MetaPhlAn vs mOTUs on the CAMI marmgCAMI2 marine sample.

## When NOT to use

- A single-value metric (e.g., F1 score or UniFrac) already answers the comparison question and no clade-level biological interpretation is needed
- Input is raw sequencing reads or binning output rather than a standardized taxonomic profile with TaxIDs that can be compared to a ground-truth profile
- A definitive, validated quantitative result on real (non-simulated) biological data is required, since TAMPA output is hypothesis-generating

## Inputs

- Input taxonomic profiler profile file(s) in CAMI-style .profile.txt format (-i, -i1; up to 3 profilers supported)
- Ground-truth taxonomic profile file (-g), e.g., data/gs_marine_short.profile.txt
- NCBI taxdump database (-d) providing the TaxID tree structure
- Taxonomic rank (positional argument, e.g., class or phylum) and sample identifier (-s, e.g., marmgCAMI2_short_read_sample_0)

## Outputs

- Annotated taxonomic tree image (png/svg/pdf via -t) at a user-specified dpi (e.g., 1600) showing per-node percentage relative abundance for each profiler versus the ground truth
- Contrast-mode tree highlighting false positives (red circles) and false negatives (blue circles) relative to the gold standard

## How to apply

Clone github.com/dkoslicki/TAMPA and build the 'CAMIViz' conda environment (python=3.7 with numpy, ete3, seaborn, pandas, matplotlib, and biom-format from the defaults, bioconda, conda-forge, and etetoolkit channels). Run src/tampa.py in normal mode, passing the profiler profile (-i, optionally a second/third since TAMPA supports up to 3 input profilers), the ground-truth profile (-g, e.g., gs_marine_short.profile.txt), and the taxonomic rank (e.g., class) as the positional argument; key parameters are -s to select a single sample (e.g., marmgCAMI2_short_read_sample_0), -k for abundance scaling (linear/log/sqrt/power), -r for output dpi (e.g., 1600), -b for the output base name, -o for the output path, and -t for file type (svg, png, pdf). The profile files and the NCBI taxdump database are used to create percentage abundance predictions per taxon, and the resulting relative abundances feed the ete3 tree-building pipeline that makes the plot. Rerun with -c True to switch to contrast mode, which highlights false positives (red circles) and false negatives (blue circles) against the gold standard and helps identify problematic subregions of large trees; -dt highlights taxa whose abundance differs by at least N percent at any rank. For large datasets condense the view with -thr (nodes above an abundance threshold) or -top (top-abundance nodes), and use -nm on monitor-less servers so the figure saves directly to file. Validate that the rendered PNG at the requested dpi depicts the correct sample and rank and matches the style of the example figures reported in the paper.

## Related tools

- **TAMPA** (Python implementation that computes percentage relative abundance per taxon and drives the ete3 tree-building pipeline in normal and contrast modes) — https://github.com/dkoslicki/TAMPA
- **ete3** (Tree-building toolkit invoked by TAMPA to render the abundance-annotated taxonomic tree plot)

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o .
```

## Evaluation signals

- The output PNG is saved at the user-defined resolution (e.g., -r 1600) for the correct sample and rank, e.g., the class-rank tree of marmgCAMI2_short_read_sample_0, matching the style of the example figures reported in the paper
- Normal-mode output shows per-node relative abundance comparisons for the profiler(s) and gold standard, with tree structure derived from NCBI taxdump TaxIDs in the profile files
- Contrast mode (-c True) marks false positives as red circles and false negatives as blue circles that correspond to taxa present only in the profiler profile or only in the gold standard
- With -thr or -top applied, only nodes above the abundance threshold or the top-abundance nodes are displayed, confirming the filter behavior
- On a monitor-less server, the plot file is written directly to the -o path only when -nm is supplied

## Limitations

- Validation was limited to simulated data; the true ground-truth distribution is only known for simulated data
- Hypotheses generated by TAMPA on real biological data require further analysis and follow-up studies for validation
- Large trees must be condensed via -thr, -top, or contrast mode to identify problematic subregions; the CAMIViz environment is pinned to python=3.7, and the Bioconda install was still waiting for a pull request to be merged

## Evidence

- [methods] These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot.: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [methods] a contrast mode, which highlights the false positives and false negatives: "a contrast mode, which highlights the false positives and false negatives"
- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [readme] TAMPA provides a "CONTRAST MODE" to better visualize the differences between the tool and gold standard.: "TAMPA provides a "CONTRAST MODE" to better visualize the differences between the tool and gold standard."
- [readme] conda create -c etetoolkit -y -n CAMIViz python=3.7 numpy  ete3  seaborn pandas matplotlib biom-format: "conda create -c etetoolkit -y -n CAMIViz python=3.7 numpy  ete3  seaborn pandas matplotlib biom-format"
- [results] This option is especially helpful when there are large trees, to identify problematic subregions: "This option is especially helpful when there are large trees, to identify problematic subregions"
- [discussion] Using simulated data was necessary in order to know the true distribution of the ground truth.: "Using simulated data was necessary in order to know the true distribution of the ground truth."
