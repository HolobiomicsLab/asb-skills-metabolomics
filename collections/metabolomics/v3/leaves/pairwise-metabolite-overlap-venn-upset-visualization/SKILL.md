---
name: pairwise-metabolite-overlap-venn-upset-visualization
description: Use when you have a filtered FT-ICR MS peak/formula table with assigned
  molecular formulas and sample-grouping metadata, and you need to quantify and visualize
  which molecular formulas are shared or unique between user-defined sample groups.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or unfiltered masses — molecular formulas must be
  assigned and peaks pre-filtered (m/z, 13C isotopes, 0.5 ppm error, minimum samples)
  before MetaboDirect can be run
- The analysis goal is quantitative abundance comparison or differential statistics
  rather than set membership of detected formulas — use the MetaboDirect statistical
  analysis or chemodiversity steps instead
- Only one sample group is defined — pairwise overlap and Venn/UpSet visualization
  require at least two groups
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Command-line pipeline whose data exploration step automatically computes pairwise
    formula overlaps and generates Venn/UpSet plots
  repo: https://github.com/Coayala/MetaboDirect
- name: UpSetR
  role: R library used to render UpSet plots of formula overlaps across grouping variables
  repo: https://github.com/Coayala/MetaboDirect
- name: ggvenn
  role: R library used to render Venn diagrams of shared/unique molecular formulas
    between groups
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Python package (used by MetaboDirect) for loading the peak/formula table and
    deriving group membership
  repo: https://github.com/Coayala/MetaboDirect
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:6c6545f7ee992624be1380477ddfb1d015a45373fcab00e8f77f7fa5205dfb4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - UpSetR
  - ggvenn
  - pandas
  - scikit-learn
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pairwise-metabolite-overlap-venn-upset-visualization

## Summary

Part of the MetaboDirect data exploration step (step iii of the 6-step pipeline): compute pairwise overlaps of detected molecular formulas between user-defined sample groups and render them as Venn and UpSet plots. This reveals which metabolites are shared or unique across conditions, e.g., phage-infected vs. control exometabolomes.

## When to use

Use when you have a filtered FT-ICR MS peak/formula table with assigned molecular formulas and sample-grouping metadata, and you need to quantify and visualize which molecular formulas are shared or unique between user-defined sample groups.

## When NOT to use

- Input is raw FT-ICR MS spectra or unfiltered masses — molecular formulas must be assigned and peaks pre-filtered (m/z, 13C isotopes, 0.5 ppm error, minimum samples) before MetaboDirect can be run
- The analysis goal is quantitative abundance comparison or differential statistics rather than set membership of detected formulas — use the MetaboDirect statistical analysis or chemodiversity steps instead
- Only one sample group is defined — pairwise overlap and Venn/UpSet visualization require at least two groups

## Inputs

- Filtered peak abundance table with assigned molecular formulas (output of MetaboDirect pre-processing/filtering step)
- Sample metadata with user-defined grouping variables (e.g., infection state, time point)

## Outputs

- Pairwise Venn diagrams of shared/unique molecular formulas between groups (PNG)
- UpSet plots of formula overlaps across all grouping variables (PNG)
- Figures written to the MetaboDirect exploration output directory

## How to apply

Load the filtered peak/formula table produced by MetaboDirect pre-processing and filtering (which has already removed masses with isotopic 13C formulas, assignment error > 0.5 ppm, and peaks below the user-defined minimum-sample threshold). From the metadata grouping variables, derive the user-defined sample groups and compute pairwise (and set-wise) overlaps of detected molecular formulas between them. Render Venn diagrams with matplotlib-venn/ggvenn and UpSet plots with UpSetR across the grouping variables, so multi-group intersections are visible beyond simple pairwise Venn circles. Write the plots as PNG files into the exploration output directory alongside the other exploration outputs (Van Krevelen, composition bar plots, NOSC/GFE/AImod/DBE violin-density plots). In the MetaboDirect application examples this step helped show that the HS2-infected exometabolome changed at 30 min post-inoculation while the HP1-infected exometabolome did not change.

## Related tools

- **MetaboDirect** (Command-line pipeline whose data exploration step automatically computes pairwise formula overlaps and generates Venn/UpSet plots) — https://github.com/Coayala/MetaboDirect
- **UpSetR** (R library used to render UpSet plots of formula overlaps across grouping variables) — https://github.com/Coayala/MetaboDirect
- **ggvenn** (R library used to render Venn diagrams of shared/unique molecular formulas between groups) — https://github.com/Coayala/MetaboDirect
- **pandas** (Python package (used by MetaboDirect) for loading the peak/formula table and deriving group membership) — https://github.com/Coayala/MetaboDirect

## Examples

```
pip install metabodirect; metabodirect -h
```

## Evaluation signals

- The exploration output directory contains Venn and UpSet plots for every user-defined group pairing, as enumerated in the MetaboDirect user's guide
- Intersection counts in the Venn/UpSet plots match the unique/shared formula lists derived from the filtered peak/formula table (e.g., after ~200 isotopic-13C masses were removed from the bacterium-phage dataset)
- All intersection cells in the UpSet plot sum to the union of detected formulas across groups
- Figures are written as PNG files in the exploration output directory without additional manual steps

## Limitations

- MetaboDirect requires molecular formula assignment to have been performed upstream; no peaks were filtered in the S. fallax data set because it was previously filtered by the original authors, so overlap results depend on prior filtering choices
- Direct injection FT-ICR MS cannot separate chemical isomers, so overlapping 'formulas' may confound distinct isobaric metabolites
- MetaboDirect requires Python (3.5+), R (4+) and Cytoscape (3.8+) with specific libraries (e.g., ggvenn, UpSetR) installed for the full pipeline to run

## Evidence

- [intro] The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [abstract] visualization examples from abstract: "pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots)"
- [readme] README dependency list including Venn/UpSet R libraries: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
- [discussion] exploration provides quick overview: "can be used to provide a quick overview of the data, upon which more in-depth analysis can be built."
