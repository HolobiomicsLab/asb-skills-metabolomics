---
name: pca-scree-biplot-generation
description: Use when FT-ICR MS metabolomic samples with assigned molecular formulas
  need ordination-based group discrimination assessment, complementing PERMANOVA and
  NMDS, using normalized peak intensities or molecular class composition vectors.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or masses without prior molecular formula assignment
  — MetaboDirect expects formula-assigned peak abundance data, not raw spectra
- Input is a small set of samples with no treatment grouping to compare (e.g., the
  4-sample S. fallax leachate set used only for exploration), where group-discriminating
  ordination is not meaningful
- The goal is chemodiversity profiling or transformation-network generation rather
  than multivariate group-discrimination assessment
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Pipeline whose statistical analysis step performs PCA with scree/biplot outputs
    on FT-ICR MS metabolomic data
  repo: https://github.com/Coayala/MetaboDirect
- name: scikit-learn
  role: Runs the PCA decomposition on molecular class composition and magnitude-averaged
    thermodynamic indices
- name: Matplotlib
  role: Renders scree plots and biplot scatter figures as .png
- name: seaborn
  role: Styles ordination scatter plots colored by treatment group
- name: pandas
  role: Loads and structures the normalized peak-intensity table for PCA input
- name: NumPy
  role: Numeric backend for PCA computation on intensity matrices
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:5c2661755e079aef5b93aae1dd4f4615a936a04f106811a9334a236dd04bc5d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  - scikit-learn
  - seaborn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# PCA scree/biplot generation for FT-ICR MS metabolomic data

## Summary

Generates principal component analysis scree plots and biplots from FT-ICR MS peak-intensity and molecular class composition data to assess whether metabolite profiles discriminate by treatment group (e.g., phage infection type) within the MetaboDirect statistical analysis step.

## When to use

Use when FT-ICR MS metabolomic samples with assigned molecular formulas need ordination-based group discrimination assessment, complementing PERMANOVA and NMDS, using normalized peak intensities or molecular class composition vectors.

## When NOT to use

- Input is raw FT-ICR MS spectra or masses without prior molecular formula assignment — MetaboDirect expects formula-assigned peak abundance data, not raw spectra
- Input is a small set of samples with no treatment grouping to compare (e.g., the 4-sample S. fallax leachate set used only for exploration), where group-discriminating ordination is not meaningful
- The goal is chemodiversity profiling or transformation-network generation rather than multivariate group-discrimination assessment

## Inputs

- Normalized peak-intensity table (peak abundances with assigned molecular formulas) as .csv
- Sample grouping metadata (e.g., phage infection type) for biplot coloring
- Molecular class composition vectors per sample
- Magnitude-averaged thermodynamic indices per sample

## Outputs

- PCA scree plot (.png)
- PCA biplot with samples colored by treatment group (.png)
- PCA component/loading tables (.csv) exported alongside PERMANOVA statistics tables

## How to apply

Load the normalized peak-intensity table (peak abundances with assigned molecular formulas) into a pandas DataFrame; normalization method should be pre-selected with MetaboDirect's test_normalization (SPANS) companion script. Run PCA with scikit-learn on two feature choices: molecular class composition vectors and magnitude-averaged thermodynamic indices, in addition to normalized intensities. Render scree plots showing variance explained per component and biplots projecting samples colored by treatment group with feature loadings, using Matplotlib/seaborn, exporting figures as .png. Interpret jointly with PERMANOVA: in the bacterium-phage data set, PERMANOVA detected significant differences among phage infection groups (p < 0.05) even though PCA/NMDS ordination could not discriminate among sample groups, indicating subtle metabolite shifts.

## Related tools

- **MetaboDirect** (Pipeline whose statistical analysis step performs PCA with scree/biplot outputs on FT-ICR MS metabolomic data) — https://github.com/Coayala/MetaboDirect
- **scikit-learn** (Runs the PCA decomposition on molecular class composition and magnitude-averaged thermodynamic indices)
- **Matplotlib** (Renders scree plots and biplot scatter figures as .png)
- **seaborn** (Styles ordination scatter plots colored by treatment group)
- **pandas** (Loads and structures the normalized peak-intensity table for PCA input)
- **NumPy** (Numeric backend for PCA computation on intensity matrices)

## Examples

```
from sklearn.decomposition import PCA; import matplotlib.pyplot as plt; p = PCA().fit(X_norm); plt.plot(p.explained_variance_ratio_); plt.savefig('scree.png')
```

## Evaluation signals

- Scree plot shows descending variance-explained per component with a defined number of retained components; cumulative explained variance is reported
- Biplot sample points are colored by the treatment grouping metadata (e.g., phage infection type) and match sample names in the input peak table
- PCA on molecular class composition and thermodynamic indices runs without error on the normalized table after formula filtering (e.g., ~200 isotopic-13C masses removed in the bacterium-phage set)
- Ordination results are consistent with PERMANOVA conclusions — significance (p < 0.05) with weak visual separation is an expected, interpretable outcome, not a failure
- Output figures are exported as .png files in the MetaboDirect output directory alongside .csv statistics tables

## Limitations

- PCA ordination could not discriminate among phage-infection sample groups despite significant PERMANOVA results, so visual ordination separation is not a reliable success criterion for subtle metabolite differences
- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression/enhancement, which can confound downstream ordination analysis
- MetaboDirect operates on data after molecular formula assignment; isotope-containing formulas must be filtered beforehand

## Evidence

- [other] Run PCA with scikit-learn on molecular class composition and magnitude-averaged thermodynamic indices to produce scree plots and biplots as .png figures.: [evidence span withheld — anchor status: not_found]
- [results] PERMANOVA significant, ordination not discriminative: [evidence span withheld — anchor status: not_found]
- [results] Normalization selection precedes analysis: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [intro] Statistical analysis is a pipeline category: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [intro] Filtering before ordination: [evidence span withheld — anchor status: not_found]
- [readme] Python dependency stack includes sklearn: [evidence span withheld — anchor status: not_found]
