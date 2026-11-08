---
name: molecular-class-composition-statistics
description: Use when normalized FT-ICR MS peak-intensity tables with assigned molecular
  formulas are available and the goal is to test group separation (e.g., infection
  treatment) via PERMANOVA and ordination (NMDS/PCA) on composition vectors.
when_to_use_negative:
- Raw FT-ICR MS spectra that have not undergone molecular formula assignment — MetaboDirect
  requires data 'after molecular formula assignment has been performed'
- Peak tables not yet normalized — run the test_normalization companion script first
  to select the best normalization method before this statistical step
- 'Goals requiring isomer-level separation: DI-MS direct injection cannot separate
  chemical isomers, so composition-level statistics cannot resolve isomeric differences'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Pipeline whose statistical analysis step runs PERMANOVA and multivariate ordination
    on normalized FT-ICR MS peak data
  repo: https://github.com/Coayala/MetaboDirect
- name: Python
  role: Language in which MetaboDirect is written and in which the dissimilarity/PERMANOVA/NMDS/PCA
    workflow is implemented
- name: pandas
  role: Holds the normalized peak-intensity table as a DataFrame for dissimilarity
    computation
- name: NumPy
  role: Numerical computation underlying matrix construction and statistics
- name: scikit-learn
  role: Runs PCA on molecular class composition and magnitude-averaged thermodynamic
    indices for scree plots and biplots
- name: Matplotlib
  role: Renders NMDS scatter plots, PCA scree plots, and biplots as .png figures
- name: seaborn
  role: Styling of the treatment-colored ordination scatter plots
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:10c5312365a2d9496e7892a0a65789d6f251f31e1d8ea31a6856dd703fb5a998
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
yes_rate: 0.6363636363636364
n_claims: 11
---

# molecular-class-composition-statistics

## Summary

Statistical analysis step of MetaboDirect that tests whether sample groupings (e.g., bacterium-phage infection type) produce significant differences in metabolite content, by running PERMANOVA on Bray-Curtis/Euclidean/Jaccard dissimilarity matrices and NMDS/PCA ordination built from normalized peak intensities and molecular class composition vectors.

## When to use

Use when normalized FT-ICR MS peak-intensity tables with assigned molecular formulas are available and the goal is to test group separation (e.g., infection treatment) via PERMANOVA and ordination (NMDS/PCA) on composition vectors.

## When NOT to use

- Raw FT-ICR MS spectra that have not undergone molecular formula assignment — MetaboDirect requires data 'after molecular formula assignment has been performed'
- Peak tables not yet normalized — run the test_normalization companion script first to select the best normalization method before this statistical step
- Goals requiring isomer-level separation: DI-MS direct injection cannot separate chemical isomers, so composition-level statistics cannot resolve isomeric differences

## Inputs

- Normalized peak-intensity table (peak abundances with assigned molecular formulas) for FT-ICR MS samples, e.g. the bacterium-phage data set of 36 samples from bacteria infected by two different phages
- Sample grouping metadata (e.g., infecting phage type / treatment) used as PERMANOVA grouping factor
- Molecular class composition vectors per sample and magnitude-averaged thermodynamic indices

## Outputs

- PERMANOVA statistics table exported as a .csv file
- NMDS scatter plots of the first two components colored by treatment group (figures)
- PCA scree plots and biplots as .png figures
- Dissimilarity matrices (Bray-Curtis, Euclidean, Jaccard) computed on normalized intensities and molecular class composition

## How to apply

Load the normalized peak-intensity table (peaks with assigned molecular formulas) into a pandas DataFrame, then compute pairwise sample dissimilarity matrices using Bray-Curtis, Euclidean, and Jaccard metrics both on normalized intensities and on molecular class composition vectors. Run PERMANOVA on these dissimilarity matrices against the sample grouping metadata and export the PERMANOVA statistics table as a .csv file — significance is judged at p < 0.05, as in the bacterium-phage data set where phage type significantly affected metabolite content (p < 0.05). Perform NMDS (via the R vegan package bundled with MetaboDirect) and render scatter plots of the first two NMDS components colored by treatment group with Matplotlib/seaborn; also run PCA with scikit-learn on molecular class composition and magnitude-averaged thermodynamic indices to produce scree plots and biplots as .png figures. Interpret ordination and PERMANOVA jointly: in the phage-host case ordination could not discriminate groups while PERMANOVA was significant, indicating subtle but real differences.

## Related tools

- **MetaboDirect** (Pipeline whose statistical analysis step runs PERMANOVA and multivariate ordination on normalized FT-ICR MS peak data) — https://github.com/Coayala/MetaboDirect
- **Python** (Language in which MetaboDirect is written and in which the dissimilarity/PERMANOVA/NMDS/PCA workflow is implemented)
- **pandas** (Holds the normalized peak-intensity table as a DataFrame for dissimilarity computation)
- **NumPy** (Numerical computation underlying matrix construction and statistics)
- **scikit-learn** (Runs PCA on molecular class composition and magnitude-averaged thermodynamic indices for scree plots and biplots)
- **Matplotlib** (Renders NMDS scatter plots, PCA scree plots, and biplots as .png figures)
- **seaborn** (Styling of the treatment-colored ordination scatter plots)

## Examples

```
metabodirect -h
```

## Evaluation signals

- PERMANOVA statistics table (.csv) exists and reports a p-value; in the reference bacterium-phage analysis phage type was significant at p < 0.05
- Dissimilarity matrices are square, symmetric, with zero diagonal, computed for all three metrics (Bray-Curtis, Euclidean, Jaccard) on both intensity and molecular-class-composition representations
- NMDS scatter plot shows samples colored by treatment group along the first two components; check whether groups visually separate against the PERMANOVA result (significant p with non-discriminating ordination indicates subtle differences)
- PCA scree plot and biplot .png files are produced from molecular class composition and magnitude-averaged thermodynamic indices
- Isotopic-carbon-containing formulas (~200 masses removed in the bacterium-phage data set) are excluded from the composition vectors before statistics

## Limitations

- Ordination (NMDS/PCA) may fail to visually discriminate sample groups even when PERMANOVA is significant, indicating differences are subtle and plots alone are not sufficient evidence
- DI-MS direct injection cannot separate chemical isomers and suffers signal suppression or enhancement that can confound downstream data analysis
- Results depend on prior filtering choices (m/z range, 13C isotopic peaks, 0.5 ppm formula assignment error, minimum sample count) and on normalization method selected via test_normalization

## Evidence

- [intro] statistical analysis step: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [results] PERMANOVA significant: "The bacterium-phage model system data set included metabolites or mass spectrometry peaks present in 36 samples from bacteria infected by two different phages"
- [results] normalization selection: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [intro] filtering prerequisites: [evidence span withheld — anchor status: not_found]
- [readme] README dependencies: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
- [results] significance threshold from source card finding: "~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon"
