---
name: grouped-composition-plot-generation
description: Use when you have a filtered peak/formula table from FT-ICR MS with per-sample
  abundances, assigned molecular formulas, and metadata grouping variables, and you
  need an overview of molecular composition and thermodynamic descriptors per group.
when_to_use_negative:
- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed
  prior to using MetaboDirect
- You need chemodiversity indices, multivariate statistics, or transformation networks
  rather than descriptive per-group composition/overview plots
- The dataset was already reduced to summary statistics with no per-sample abundance
  and formula table to facet by group
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Pipeline whose data-exploration step implements the grouped composition plot
    generation (Van Krevelen, composition, violin/density, Venn/upset outputs)
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loads the peak/formula table and computes derived descriptors (H/C, O/C, NOSC,
    GFE, AImod, DBE) per group
- name: NumPy
  role: Numeric computation of molecular descriptors used as plot coordinates and
    violin/density variables
- name: Matplotlib
  role: Rendering of Van Krevelen diagrams, composition bar plots, and violin/density
    figures
- name: seaborn
  role: Grouped/facetted statistical plotting (violin and density plots of NOSC, GFE,
    AImod, DBE)
- name: statsmodels
  role: Tukey HSD test to compute significance letters overlaid on descriptor violin/density
    plots (Python dependency listed in README)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:f32ec803847d9207be52b5a2f8226823f33556a16f30be49f5a2120600d18a17
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# grouped-composition-plot-generation

## Summary

Generates the MetaboDirect data-exploration figure set — Van Krevelen diagrams, elemental and molecular class composition bar plots, NOSC/GFE/AImod/DBE violin-density plots with Tukey HSD letters, and pairwise Venn/upset plots — facetted by user-defined sample groups for direct-injection FT-ICR MS metabolomic data.

## When to use

Use when you have a filtered peak/formula table from FT-ICR MS with per-sample abundances, assigned molecular formulas, and metadata grouping variables, and you need an overview of molecular composition and thermodynamic descriptors per group.

## When NOT to use

- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed prior to using MetaboDirect
- You need chemodiversity indices, multivariate statistics, or transformation networks rather than descriptive per-group composition/overview plots
- The dataset was already reduced to summary statistics with no per-sample abundance and formula table to facet by group

## Inputs

- Filtered peak abundance table with assigned molecular formulas from FT-ICR MS (post MetaboDirect pre-processing/filtering, ≤0.5 ppm formula error)
- Metadata file with user-defined grouping variables (e.g., infection/time-point treatments)
- Derived per-formula descriptors: H/C, O/C, NOSC, GFE, AImod, DBE (computed within the step)

## Outputs

- Van Krevelen diagrams (H/C vs O/C) per group (PNG)
- Elemental and molecular class composition bar plots per group (PNG)
- NOSC, GFE, AImod, and DBE violin/density plots annotated with Tukey HSD significance letters (PNG)
- Pairwise Venn and upset plots of formula overlaps across groups (PNG)

## How to apply

Load the filtered peak/formula table produced by MetaboDirect pre-processing (after m/z, 13C isotope, and 0.5 ppm formula-error filtering) keeping per-sample abundances, assigned formulas, and grouping variables from the metadata. Compute derived molecular descriptors per formula — H/C and O/C ratios for Van Krevelen coordinates, plus NOSC, GFE, AImod, and DBE — using the pandas/NumPy formulas implemented in MetaboDirect. Render Van Krevelen diagrams and elemental/molecular class composition bar plots with Matplotlib and seaborn, colored/facetted by the user-defined groups, then generate violin and density plots of NOSC, GFE, AImod, and DBE per group with statsmodels/scipy Tukey HSD significance letters overlaid to mark statistically different group medians. Finally compute pairwise overlaps of detected formulas between groups and render Venn and upset plots, writing all figures as PNG files to the exploration output directory. In the showcase applications this revealed the HS2-infected exometabolome changing at 30 min post-inoculation before reverting, while HP1-infected samples were unchanged.

## Related tools

- **MetaboDirect** (Pipeline whose data-exploration step implements the grouped composition plot generation (Van Krevelen, composition, violin/density, Venn/upset outputs)) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loads the peak/formula table and computes derived descriptors (H/C, O/C, NOSC, GFE, AImod, DBE) per group)
- **NumPy** (Numeric computation of molecular descriptors used as plot coordinates and violin/density variables)
- **Matplotlib** (Rendering of Van Krevelen diagrams, composition bar plots, and violin/density figures)
- **seaborn** (Grouped/facetted statistical plotting (violin and density plots of NOSC, GFE, AImod, DBE))
- **statsmodels** (Tukey HSD test to compute significance letters overlaid on descriptor violin/density plots (Python dependency listed in README))

## Examples

```
metabodirect -h
```

## Evaluation signals

- Exploration output directory contains a Van Krevelen plot, elemental and molecular class composition bar plots, NOSC/GFE/AImod/DBE violin-density plots with Tukey letters, and pairwise Venn/upset plots for every defined group, as enumerated in the MetaboDirect user's guide
- All figures are written as PNG files to the exploration output directory
- H/C and O/C coordinates fall in chemically plausible ranges for assigned CHO/CHNO(S)P formulas
- Tukey HSD letters on violin plots correctly group medians: groups sharing a letter are not significantly different
- Venn/upset plot intersection counts match pairwise overlaps of detected formula lists computed from the input table

## Limitations

- Molecular formula assignment must be performed before MetaboDirect; the pipeline accepts only peak abundance plus assigned formula data
- Direct injection FT-ICR MS cannot separate chemical isomers, and signal suppression/enhancement can confound downstream data analysis, limiting interpretation of composition plots
- Grouping depends on user-supplied metadata variables; incorrect or absent grouping variables yield meaningless facetted plots

## Evidence

- [abstract] pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots): "pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots)"
- [abstract] MetaboDirect packages: [evidence span withheld — anchor status: not_found]
- [intro] data exploration as a pipeline step: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [readme] README CLI usage: [evidence span withheld — anchor status: external_doc]
