---
name: nmds-ordination-visualization
description: Use when a normalized peak-intensity table (peaks with assigned molecular
  formulas) plus sample grouping metadata need ordination to visually test whether
  treatment groups separate, and when PERMANOVA significance must be interpreted against
  ordination plots.
when_to_use_negative:
- Input is raw FT-ICR MS spectra — MetaboDirect expects data after molecular formula
  assignment has already been performed
- The goal is annotation/pathway mapping rather than ordination — use the KEGG (KEGGREST)
  query step or transformation-network step instead
- Only two samples are available (e.g., the 4-sample S. fallax leachate data set)
  where group-level ordination and PERMANOVA lack power
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: command-line pipeline whose statistical-analysis step performs PERMANOVA,
    NMDS, and PCA on the peak table
  repo: https://github.com/Coayala/MetaboDirect
- name: vegan
  role: R package (MetaboDirect dependency) used for dissimilarity-based ordination
    and PERMANOVA
- name: scikit-learn
  role: PCA computation on molecular class composition and thermodynamic indices
- name: seaborn
  role: rendering NMDS scatter plots colored by treatment group
- name: Matplotlib
  role: figure generation for NMDS scatter plots, PCA scree plots, and biplots
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:ec93ad84ed958229a63d13cdcf85368f6f597ecdd8acaf8604d8d4b05c60edfe
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
  - vegan
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9130434782608695
n_claims: 23
---

# nmds-ordination-visualization

## Summary

Ordination-based visualization (NMDS and PCA scree/biplots) of FT-ICR MS metabolomic profiles, paired with PERMANOVA on Bray-Curtis/Euclidean/Jaccard dissimilarity matrices, to assess whether treatment groups (e.g., different infecting phages) differ in metabolite content.

## When to use

Use when a normalized peak-intensity table (peaks with assigned molecular formulas) plus sample grouping metadata need ordination to visually test whether treatment groups separate, and when PERMANOVA significance must be interpreted against ordination plots.

## When NOT to use

- Input is raw FT-ICR MS spectra — MetaboDirect expects data after molecular formula assignment has already been performed
- The goal is annotation/pathway mapping rather than ordination — use the KEGG (KEGGREST) query step or transformation-network step instead
- Only two samples are available (e.g., the 4-sample S. fallax leachate data set) where group-level ordination and PERMANOVA lack power

## Inputs

- Normalized peak-intensity table (peak abundances with assigned molecular formulas) from FT-ICR MS data
- Sample metadata / grouping file defining treatment groups (e.g., phage infection type)
- Molecular class composition vectors and magnitude-averaged thermodynamic indices per sample (for PCA)

## Outputs

- NMDS scatter plots of the first two components colored by treatment group (.png)
- PCA scree plots and biplots (.png)
- PERMANOVA statistics table (.csv)

## How to apply

Load the normalized peak-intensity table (peak abundances with assigned molecular formulas) into a pandas DataFrame. Compute pairwise sample dissimilarity matrices using Bray-Curtis, Euclidean, and Jaccard metrics on the normalized intensities and on molecular class composition vectors, then run PERMANOVA of these matrices against the sample grouping metadata and export the statistics table as .csv. Perform non-metric multidimensional scaling and render scatter plots of the first two NMDS components colored by treatment group (e.g., phage infection type) using Matplotlib/seaborn; run PCA with scikit-learn on molecular class composition and magnitude-averaged thermodynamic indices to produce scree plots and biplots as .png figures. Interpret ordination together with PERMANOVA: in the bacterium-phage data set, phage type produced a significant difference in metabolite content (p < 0.05) even though NMDS/PCA could not discriminate among groups, indicating the differences are subtle. Within MetaboDirect this is the pipeline's statistical-analysis step, runnable as part of its single-line automated framework.

## Related tools

- **MetaboDirect** (command-line pipeline whose statistical-analysis step performs PERMANOVA, NMDS, and PCA on the peak table) — https://github.com/Coayala/MetaboDirect
- **vegan** (R package (MetaboDirect dependency) used for dissimilarity-based ordination and PERMANOVA)
- **scikit-learn** (PCA computation on molecular class composition and thermodynamic indices)
- **seaborn** (rendering NMDS scatter plots colored by treatment group)
- **Matplotlib** (figure generation for NMDS scatter plots, PCA scree plots, and biplots)

## Examples

```
Rscript -e 'library(vegan); nmds <- metaMDS(t(peak_table), distance="bray", k=2); plot(nmds, type="t")'
```

## Evaluation signals

- NMDS scatter plot shows the first two components with points colored by treatment group (e.g., phage vs. no-phage / phage 1 vs. phage 2)
- PERMANOVA statistics .csv reports p < 0.05 for the grouping factor on the bacterium-phage data set, consistent with the published finding
- Dissimilarity matrices computed for all three metrics (Bray-Curtis, Euclidean, Jaccard) and on both normalized intensities and molecular class composition vectors
- PCA scree plot reports variance explained per component and biplot overlays samples and molecular-class/thermodynamic-index loadings
- All figures exported as .png and statistics as .csv without long compute time (pipeline designed to run with a single line of code)

## Limitations

- Ordination (NMDS/PCA) could not discriminate among sample groups even when PERMANOVA was significant (p < 0.05), because metabolite differences were subtle — visual non-separation does not imply absence of effect
- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression/enhancement that can confound downstream analysis
- Requires R (4+), Python (3.5+), and Cytoscape (3.8+) with packages including vegan and scikit-learn installed; ~200 masses were filtered from the bacterium-phage data set due to isotopic-carbon formulas before analysis

## Evidence

- [results] PERMANOVA on the bacterium-phage data set showed that the type of phage infecting the cells produces a significant difference in the metabolite content: [evidence span withheld — anchor status: not_found]
- [abstract] pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams: "an open-source, command-line-based pipeline for the analysis (e.g., chemodiversity analysis, multivariate statistics), visualization (e.g., Van Krevelen diagrams"
- [intro] The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [readme] MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
