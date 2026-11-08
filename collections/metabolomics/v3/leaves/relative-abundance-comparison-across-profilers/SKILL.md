---
name: relative-abundance-comparison-across-profilers
description: Use when benchmarking CAMI-style taxonomic profiles where single-value
  metrics (F1 score, L1 norm, UniFrac) look equivalent across profilers, and you need
  to see which clades at class/phylum rank drive abundance differences versus ground
  truth.
when_to_use_negative:
- Input is raw reads or assemblies rather than standardized taxonomic profile files
  — TAMPA consumes profiler profiles plus the NCBI taxdump, not sequencing data.
- The analysis targets amplicon microbial datasets; the paper notes metacoder serves
  amplicon data, while TAMPA is designed for WGS metagenomic taxonomic profiles.
- Only an overall accuracy ranking is needed (e.g., a single F1 score) with no need
  to inspect which clades contribute to the metric values.
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3293
tools:
- name: TAMPA
  role: Computes percentage relative abundance per taxon from profiler and ground-truth
    profiles plus the NCBI taxdump, then renders normal-mode and contrast-mode taxonomic
    tree comparisons
  repo: https://github.com/dkoslicki/TAMPA
- name: ete3
  role: Tree-building toolkit invoked by TAMPA to render the abundance-annotated taxonomic
    tree plot
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA has been integrated,
    adding tree-based abundance comparison to profiler evaluation
  repo: https://github.com/CAMI-challenge/OPAL
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:5393940dfd3e6219e35ed41dcc5038f72f530e786abf00a507686f7631b9aabf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA
  tools:
  - OPAL
  - TAMPA
  - ete3
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# relative-abundance-comparison-across-profilers

## Summary

Compares percentage relative abundances of taxonomic profiler outputs (e.g., MetaPhlAn vs mOTUs on the CAMI marmgCAMI2 marine sample) against a gold-standard profile using TAMPA, which renders the comparison on an ete3 taxonomic tree so clade-level differences missed by single-value metrics like the F1 score become visible.

## When to use

Use when benchmarking CAMI-style taxonomic profiles where single-value metrics (F1 score, L1 norm, UniFrac) look equivalent across profilers, and you need to see which clades at class/phylum rank drive abundance differences versus ground truth.

## When NOT to use

- Input is raw reads or assemblies rather than standardized taxonomic profile files — TAMPA consumes profiler profiles plus the NCBI taxdump, not sequencing data.
- The analysis targets amplicon microbial datasets; the paper notes metacoder serves amplicon data, while TAMPA is designed for WGS metagenomic taxonomic profiles.
- Only an overall accuracy ranking is needed (e.g., a single F1 score) with no need to inspect which clades contribute to the metric values.

## Inputs

- Taxonomic profiler output profile file in standardized CAMI-style format (e.g., data/mad_yalow_0.profile.txt); up to 3 profilers supported via -i/-i1
- Ground-truth taxonomic profile (e.g., data/gs_marine_short.profile.txt) passed via -g
- NCBI taxdump database as the default taxonomy source (or a custom dump via -d)
- Sample identifier of interest selected with -s (e.g., marmgCAMI2_short_read_sample_0)

## Outputs

- Taxonomic tree plot (PNG at user-set dpi, e.g., 1600) showing per-taxon relative abundance, rendered by the ete3 tree-building pipeline
- Contrast-mode tree image with false-positive taxa as red circles and false-negative taxa as blue circles relative to the gold standard
- Optional L1 error plot when -l is specified

## How to apply

Clone github.com/dkoslicki/TAMPA and create the 'CAMIViz' conda environment (python=3.7 with numpy, ete3, seaborn, pandas, matplotlib, biom-format from the defaults, bioconda, conda-forge, and etetoolkit channels). TAMPA reads the profiler profile file (-i, e.g., data/mad_yalow_0.profile.txt) and ground-truth profile (-g, e.g., data/gs_marine_short.profile.txt), and together with the NCBI taxdump database computes percentage abundance predictions for each taxon; pass the taxonomic rank (e.g., class), sample of interest (-s marmgCAMI2_short_read_sample_0), linear scaling (-k linear), and resolution (-r 1600 dpi) on the command line. Run normal mode first (-c False) to display relative abundance across up to 3 input profilers on one tree, then rerun with -c True so contrast mode draws false-positive taxa as red circles and false-negative taxa as blue circles against the gold standard (true positives white, remaining taxa on a white-to-green relative-error gradient). For large trees condense the view with -thr (minimum abundance threshold) or -top (top-abundance nodes), optionally add the L1 error plot with -l and normalize node weights with -n. The rationale is that single-value metrics fail to capture biological differences: on the marine CAMI second-round sample this comparison exposed MetaPhlAn missing Tenericutes and Planctomycetes and a considerable Firmicutes abundance gap versus mOTUs.

## Related tools

- **TAMPA** (Computes percentage relative abundance per taxon from profiler and ground-truth profiles plus the NCBI taxdump, then renders normal-mode and contrast-mode taxonomic tree comparisons) — https://github.com/dkoslicki/TAMPA
- **ete3** (Tree-building toolkit invoked by TAMPA to render the abundance-annotated taxonomic tree plot)
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA has been integrated, adding tree-based abundance comparison to profiler evaluation) — https://github.com/CAMI-challenge/OPAL

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o .
```

## Evaluation signals

- Output PNG exists at the requested resolution (-r 1600 dpi) for the specified rank and sample, i.e., the class-level tree of marmgCAMI2_short_read_sample_0.
- Visual inspection of contrast mode confirms false-positive taxa appear as red circles and false-negative taxa as blue circles, matching the paper's contrast-mode specification.
- Normal-mode tree displays side-by-side relative abundance for the profiles; known marine-sample differences (MetaPhlAn missing Tenericutes and Planctomycetes; Firmicutes abundance gap vs mOTUs) are visible.
- Spot-check a taxon's plotted abundance against its value in the input profile files, confirming percentage abundances were derived from profiles plus the NCBI taxdump before tree construction.
- Documented filters behave as specified: -thr/-top reduce displayed nodes, -l adds the L1 error plot, and -n makes node weights sum to one.

## Limitations

- TAMPA output is hypothesis-generating: the paper states further analysis and follow-up studies are required to validate hypotheses generated on real biological data.
- The true ground-truth distribution is only known for simulated data (e.g., CAMI), so contrast mode against a validated gold standard is limited to such datasets.
- Up to 3 input profilers are supported, and very large trees may require -thr/-top condensation or contrast mode to identify problematic subregions.

## Evidence

- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [methods] These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot.: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [abstract] single metrics, such as the F1 score, fail to capture the biological differences between the datasets: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [results] TAMPA can support up to 3 input profilers at once, illuminating differences in their relative abundances: "TAMPA can support up to 3 input profilers at once, illuminating differences in their relative abundances"
- [discussion] While TAMPA can be used for hypothesis generation on real biological data, further analysis and follow-up studies will be required to validate the hypotheses generated by TAMPA.: "While TAMPA can be used for hypothesis generation on real biological data, further analysis and follow-up studies will be required to validate the hypotheses generated by TAMPA."
- [readme] python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o .: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o ."
- [readme] The contrast mode can be activated by setting the parameter c to True: "The contrast mode can be activated by setting the parameter c to True"
