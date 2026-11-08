---
name: fticr-ms-van-krevelen-visualization
description: Use when you have a filtered FT-ICR MS peak abundance table with assigned
  molecular formulas plus sample-group metadata, and you need visual overview plots
  (Van Krevelen, class composition, thermodynamic indices, group overlaps) per group.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or masses without assigned molecular formulas — molecular
  formula assignment (e.g., MFAssignR/Formularity) must be performed prior to MetaboDirect
- Peak table has not been filtered (13C isotopic masses, m/z bounds, 0.5 ppm assignment
  error, minimum sample threshold) — run MetaboDirect pre-processing/filtering first
- The goal is deep statistical or pathway-level analysis rather than a quick overview;
  this step 'can be used to provide a quick overview of the data, upon which more
  in-depth analysis can be built'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboDirect
  role: Command-line pipeline whose data exploration step orchestrates descriptor
    computation and figure generation
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Data handling and computation of H/C, O/C, NOSC, GFE, AImod, DBE descriptors
    from formula table
- name: NumPy
  role: Numerical computation of molecular descriptors
- name: Matplotlib
  role: Rendering Van Krevelen diagrams, composition bar plots, and violin/density
    figures
- name: seaborn
  role: Statistical plotting of descriptor distributions per group
- name: statsmodels
  role: Tukey HSD test producing significance letters overlaid on violin/density plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:167126c73363ac4a36e5555a96feb91b612273ec4f82b48e6157f26584a94c54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - pandas
  - scikit-learn
  - seaborn
  - statsmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fticr-ms-van-krevelen-visualization

## Summary

Generates Van Krevelen diagrams (H/C vs O/C), elemental and molecular class composition plots, NOSC/GFE/AImod/DBE violin-density plots with Tukey HSD letters, and pairwise Venn/upset overlap plots from FT-ICR MS peak/formula tables as part of the MetaboDirect data exploration step.

## When to use

Use when you have a filtered FT-ICR MS peak abundance table with assigned molecular formulas plus sample-group metadata, and you need visual overview plots (Van Krevelen, class composition, thermodynamic indices, group overlaps) per group.

## When NOT to use

- Input is raw FT-ICR MS spectra or masses without assigned molecular formulas — molecular formula assignment (e.g., MFAssignR/Formularity) must be performed prior to MetaboDirect
- Peak table has not been filtered (13C isotopic masses, m/z bounds, 0.5 ppm assignment error, minimum sample threshold) — run MetaboDirect pre-processing/filtering first
- The goal is deep statistical or pathway-level analysis rather than a quick overview; this step 'can be used to provide a quick overview of the data, upon which more in-depth analysis can be built'

## Inputs

- Filtered FT-ICR MS peak abundance table with assigned molecular formulas (output of MetaboDirect pre-processing/filtering)
- Sample metadata file with user-defined grouping variables
- Filter parameters (m/z range, 13C isotopic peak removal, 0.5 ppm formula assignment error tolerance, minimum sample count threshold) already applied upstream

## Outputs

- Van Krevelen diagrams (H/C vs O/C) per group as PNG files
- Elemental and molecular class composition bar plots per group as PNG files
- Violin and density plots of NOSC, GFE, AImod, and DBE per group with Tukey HSD significance letters
- Pairwise Venn (matplotlib-venn / ggvenn) and upset plots (UpSetR) of formula overlaps across groups
- Exploration output directory containing all figures enumerated in the MetaboDirect user's guide

## How to apply

Load the filtered peak/formula table produced by MetaboDirect pre-processing and filtering, keeping per-sample abundances, assigned formulas, and metadata grouping variables. Compute derived molecular descriptors per formula (H/C and O/C for Van Krevelen coordinates, plus NOSC, GFE, AImod, and DBE) using the pandas/NumPy implementations in MetaboDirect. Plot Van Krevelen diagrams and elemental/molecular class composition bar plots with Matplotlib and seaborn, colored or facetted by the user-defined grouping variables; generate violin and density plots of NOSC, GFE, AImod, and DBE per group and overlay Tukey HSD significance letters from statsmodels/scipy to mark statistically different group medians. Compute pairwise formula overlaps between groups and render Venn and upset plots, writing all figures as PNGs to the exploration output directory. The whole pipeline is designed to run with a single line of code, e.g. via the `metabodirect` CLI.

## Related tools

- **MetaboDirect** (Command-line pipeline whose data exploration step orchestrates descriptor computation and figure generation) — https://github.com/Coayala/MetaboDirect
- **pandas** (Data handling and computation of H/C, O/C, NOSC, GFE, AImod, DBE descriptors from formula table)
- **NumPy** (Numerical computation of molecular descriptors)
- **Matplotlib** (Rendering Van Krevelen diagrams, composition bar plots, and violin/density figures)
- **seaborn** (Statistical plotting of descriptor distributions per group)
- **statsmodels** (Tukey HSD test producing significance letters overlaid on violin/density plots)

## Examples

```
metabodirect -h
```

## Evaluation signals

- Exploration output directory contains a Van Krevelen plot, elemental/molecular class bar plots, NOSC/GFE/AImod/DBE violin-density plots with Tukey letters, and pairwise Venn/upset plots for every defined group, as enumerated in the MetaboDirect user's guide
- Computed H/C and O/C ratios reproduce from element counts of assigned molecular formulas; NOSC, GFE, AImod, and DBE values match the pandas/NumPy formulas implemented in MetaboDirect
- Tukey HSD letters partition groups whose medians differ significantly; groups sharing letters are not statistically separated
- All figures written as PNG files without errors from the single-line pipeline run

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers, so Van Krevelen/composition plots aggregate isobaric and isomeric species under one formula
- Signal suppression or enhancement in direct injection MS can confound downstream data analysis, biasing apparent group compositions
- Peaks must already carry assigned molecular formulas; formula assignment is performed prior to MetaboDirect

## Evidence

- [abstract] pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots): "pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots)"
- [intro] six major steps including data exploration: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [intro] filtering criteria applied before exploration: [evidence span withheld — anchor status: not_found]
- [abstract] single-line automated figure generation: "MetaboDirect is superior in that it requires a single line of code to launch a fully automated framework for the generation and visualization of a wide range of plots, with minimal coding experience"
- [readme] README dependency list including plotting and stats libraries: "- argparse
- numpy
- pandas
- seaborn
- more-itertools
- py4cytoscape
- statsmodels"
- [discussion] overview purpose of exploration outputs: "can be used to provide a quick overview of the data, upon which more in-depth analysis can be built."
