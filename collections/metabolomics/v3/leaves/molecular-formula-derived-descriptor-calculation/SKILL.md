---
name: molecular-formula-derived-descriptor-calculation
description: Use when you have a filtered FT-ICR MS peak/formula table with per-sample
  abundances and assigned molecular formulas, and need derived thermodynamic/structural
  descriptors to plot Van Krevelen diagrams, violin plots of NOSC/GFE/AImod/DBE, or
  compare sample groups.
when_to_use_negative:
- Input is raw FT-ICR MS spectra that have not undergone molecular formula assignment
  — MetaboDirect requires formulas to be assigned beforehand
- Peak table has not yet been filtered (13C isotopic masses, m/z range, assignment
  error still present), since unfiltered formulas will skew descriptor distributions
- The analysis goal is chemodiversity or transformation-network generation rather
  than descriptor-based molecular characterization — those are separate MetaboDirect
  steps
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboDirect
  role: Pipeline implementing the descriptor-calculation formulas and the downstream
    exploration figures
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Tabular computation of per-formula descriptors from the peak/formula table
- name: NumPy
  role: Numerical computation underlying descriptor formulas
- name: Matplotlib
  role: Rendering Van Krevelen diagrams and descriptor violin/density plots
- name: seaborn
  role: Statistical visualization of descriptor distributions per group
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:7a342d1909e9886caea84d028ca66a4289bff492b5c393a111356c62eb3828ae
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# molecular-formula-derived-descriptor-calculation

## Summary

Compute molecular descriptors (H/C and O/C ratios, NOSC, GFE, AImod, DBE) from assigned molecular formulas in FT-ICR MS peak tables using the pandas/NumPy formulas implemented in MetaboDirect, to characterize the molecular and thermodynamic properties of metabolomic samples.

## When to use

Use when you have a filtered FT-ICR MS peak/formula table with per-sample abundances and assigned molecular formulas, and need derived thermodynamic/structural descriptors to plot Van Krevelen diagrams, violin plots of NOSC/GFE/AImod/DBE, or compare sample groups.

## When NOT to use

- Input is raw FT-ICR MS spectra that have not undergone molecular formula assignment — MetaboDirect requires formulas to be assigned beforehand
- Peak table has not yet been filtered (13C isotopic masses, m/z range, assignment error still present), since unfiltered formulas will skew descriptor distributions
- The analysis goal is chemodiversity or transformation-network generation rather than descriptor-based molecular characterization — those are separate MetaboDirect steps

## Inputs

- Filtered peak abundance table with assigned molecular formulas (output of MetaboDirect pre-processing/filtering, after removing 13C isotopic masses and peaks failing the 0.5 ppm assignment-error threshold)
- Sample metadata with grouping variables (e.g., phage treatment, time point)

## Outputs

- Per-formula derived descriptor table: H/C, O/C, NOSC, GFE, AImod, DBE
- Descriptor columns feeding Van Krevelen diagrams and NOSC/GFE/AImod/DBE violin-density plots with Tukey HSD letters

## How to apply

Load the filtered peak/formula table produced by the MetaboDirect pre-processing and filtering step (peaks already filtered by m/z, 13C isotopic presence, 0.5 ppm formula-assignment error, and minimum sample threshold). For each assigned molecular formula, compute the derived descriptors using the pandas/NumPy formulas implemented in MetaboDirect: H/C and O/C ratios (used as Van Krevelen coordinates), NOSC (nominal oxidation state of carbon), GFE (gibbs free energy), AImod (modified aromaticity index), and DBE (double bond equivalents). These descriptors are the basis for downstream exploration figures — Van Krevelen diagrams, elemental and molecular class composition bar plots, and violin/density plots per sample group with Tukey HSD significance letters. Validate by checking that computed values are finite for all formulas and that descriptor distributions differ across the user-defined grouping variables as expected.

## Related tools

- **MetaboDirect** (Pipeline implementing the descriptor-calculation formulas and the downstream exploration figures) — https://github.com/Coayala/MetaboDirect
- **pandas** (Tabular computation of per-formula descriptors from the peak/formula table)
- **NumPy** (Numerical computation underlying descriptor formulas)
- **Matplotlib** (Rendering Van Krevelen diagrams and descriptor violin/density plots)
- **seaborn** (Statistical visualization of descriptor distributions per group)

## Examples

```
metabodirect -i filtered_peak_table.csv -m metadata.csv -o exploration_output -g phage_treatment,time_point
```

## Evaluation signals

- Exploration output directory contains a Van Krevelen plot, elemental/molecular class bar plots, and NOSC/GFE/AImod/DBE violin-density plots with Tukey letters for every defined group
- All computed descriptor values are finite (no NaNs from malformed formulas)
- H/C and O/C values fall within chemically plausible ranges consistent with the assigned formula element counts
- Descriptor distributions are facetted/colored correctly by the user-defined grouping variables
- Pipeline runs from a single command line invocation without long compute times, as designed for MetaboDirect

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression/enhancement that can confound downstream analysis
- Descriptors are only as reliable as the upstream formula assignment; formulas containing isotopic carbon (~200 masses removed in the bacterium-phage data set) must be filtered out first
- S. fallax data set required no filtering because it was previously filtered by the original authors — descriptor quality depends on the provenance of the input table

## Evidence

- [results] Compute the derived descriptors for each formula (H/C and O/C ratios for the Van Krevelen coordinates, NOSC, GFE, AImod, and DBE) using the pandas/NumPy formulas implemented in MetaboDirect.: [evidence span withheld — anchor status: not_found]
- [intro] detected peaks are filtered by their m/z values (based on the user's input), isotopic presence (13C peaks), error in formula assignment (0.5 ppm): [evidence span withheld — anchor status: not_found]
- [abstract] MetaboDirect is written in Python and uses a wide variety of packages, including pandas, NumPy, Matplotlib, seaborn, scikit-learn, and NetworkX: [evidence span withheld — anchor status: not_found]
- [abstract] pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots): "pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots)"
- [intro] The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
