---
name: dissimilarity-matrix-computation-ft-icr-ms
description: Use when you have a normalized FT-ICR MS peak-intensity table with assigned
  molecular formulas and need pairwise sample dissimilarities (Bray-Curtis, Euclidean,
  Jaccard) as input to PERMANOVA group testing or NMDS/PCA ordination.
when_to_use_negative:
- Input is still raw FT-ICR MS spectra — MetaboDirect requires that molecular formula
  assignment has already been performed before its analysis steps
- Peak table is not normalized — run the test_normalization companion script first
  to select a normalization method before computing dissimilarities
- The data are not direct-injection FT-ICR MS peak/formula tables (e.g., LC-MS feature
  tables or other omics modalities outside the pipeline's scope)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboDirect
  role: Pipeline whose statistical analysis step provides multivariate statistics
    (PERMANOVA, NMDS, PCA) on FT-ICR MS metabolomic profiles
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loads and holds the normalized peak-intensity table and molecular class composition
    vectors as DataFrames
- name: scikit-learn
  role: Runs PCA on molecular class composition and thermodynamic indices for scree
    plots and biplots
- name: vegan (R)
  role: R dependency of MetaboDirect used for community-ecology style dissimilarity/ordination
    analysis underlying the statistical step
- name: Matplotlib
  role: Renders NMDS scatter plots, PCA scree plots, and biplots as .png figures
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:9247a81c55add6422dd23fbcee47b3f87d81ea22210c5327b0a12a9060653000
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
  - vegan (R)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# dissimilarity-matrix-computation-ft-icr-ms

## Summary

Computes pairwise sample dissimilarity matrices (Bray-Curtis, Euclidean, Jaccard) from normalized FT-ICR MS peak-abundance tables with assigned molecular formulas, as implemented in MetaboDirect's statistical analysis step, to support PERMANOVA testing and NMDS/PCA ordination.

## When to use

Use when you have a normalized FT-ICR MS peak-intensity table with assigned molecular formulas and need pairwise sample dissimilarities (Bray-Curtis, Euclidean, Jaccard) as input to PERMANOVA group testing or NMDS/PCA ordination.

## When NOT to use

- Input is still raw FT-ICR MS spectra — MetaboDirect requires that molecular formula assignment has already been performed before its analysis steps
- Peak table is not normalized — run the test_normalization companion script first to select a normalization method before computing dissimilarities
- The data are not direct-injection FT-ICR MS peak/formula tables (e.g., LC-MS feature tables or other omics modalities outside the pipeline's scope)

## Inputs

- Normalized peak-intensity table (peak abundances with assigned molecular formulas) as .csv
- Sample grouping metadata (e.g., infecting phage type / treatment labels)
- Optionally, molecular class composition vectors and magnitude-averaged thermodynamic indices for PCA

## Outputs

- Pairwise sample dissimilarity matrices (Bray-Curtis, Euclidean, Jaccard)
- PERMANOVA statistics table exported as .csv
- NMDS scatter plots of the first two components colored by treatment group (.png)
- PCA scree plots and biplots (.png)

## How to apply

Load the normalized peak-intensity table (peak abundances with assigned molecular formulas) into a pandas DataFrame; normalization method should be chosen beforehand with the MetaboDirect 'test_normalization' companion script (SPANS). Compute pairwise sample dissimilarity matrices using Bray-Curtis, Euclidean, and Jaccard metrics on (i) the normalized peak intensities and (ii) molecular class composition vectors. Use these matrices as input to PERMANOVA against the sample grouping metadata (e.g., which phage infected the bacterial cells) and to NMDS/PCA ordination; export the PERMANOVA statistics table as .csv and ordination figures (NMDS scatters colored by treatment group, PCA scree plots and biplots) as .png. Rationale: in the MetaboDirect bacterium-phage application, PERMANOVA detected a significant metabolite-content difference by infecting phage type (p < 0.05) even though NMDS/PCA ordination could not visually discriminate the groups, so significance testing on dissimilarity matrices catches subtle differences that plots miss.

## Related tools

- **MetaboDirect** (Pipeline whose statistical analysis step provides multivariate statistics (PERMANOVA, NMDS, PCA) on FT-ICR MS metabolomic profiles) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loads and holds the normalized peak-intensity table and molecular class composition vectors as DataFrames)
- **scikit-learn** (Runs PCA on molecular class composition and thermodynamic indices for scree plots and biplots)
- **vegan (R)** (R dependency of MetaboDirect used for community-ecology style dissimilarity/ordination analysis underlying the statistical step)
- **Matplotlib** (Renders NMDS scatter plots, PCA scree plots, and biplots as .png figures)

## Examples

```
from sklearn.metrics import pairwise_distances; D = pairwise_distances(X_norm, metric='braycurtis')
```

## Evaluation signals

- Dissimilarity matrices are square, symmetric, with zero diagonal and off-diagonal values in the metric's expected range (e.g., Bray-Curtis and Jaccard in [0, 1])
- PERMANOVA statistics table (.csv) reports a significant grouping effect (p < 0.05) for the phage-type factor on the bacterium-phage data set
- NMDS/PCA ordination plots are produced as .png and, per the reference result, may fail to visually separate groups even when PERMANOVA is significant — do not treat visual non-separation as a failure
- Input table was normalized via the method selected by the test_normalization script, not raw peak abundances
- Metrics are computed on both normalized intensities and molecular class composition vectors, and all three metrics (Bray-Curtis, Euclidean, Jaccard) are reported

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement that can confound downstream data analysis
- Differences detected by PERMANOVA can be subtle and invisible in NMDS/PCA ordination, so ordination alone is insufficient for judging group separation
- Compute time of the main pipeline grows with the number of samples and average number of metabolites with assigned molecular formulas

## Evidence

- [abstract] MetaboDirect is written in Python and uses a wide variety of packages, including pandas, NumPy, Matplotlib, seaborn, scikit-learn, and NetworkX: [evidence span withheld — anchor status: not_found]
- [intro] MetaboDirect was designed to facilitate data exploration, data visualization, chemodiversity, and statistical analysis of metabolomic profiles, as well as the generation of transformation networks: "MetaboDirect was designed to facilitate data exploration, data visualization, chemodiversity, and statistical analysis of metabolomic profiles, as well as the generation of transformation networks"
- [results] The bacterium-phage model system data set included metabolites or mass spectrometry peaks present in 36 samples from bacteria infected by two different phages: "The bacterium-phage model system data set included metabolites or mass spectrometry peaks present in 36 samples from bacteria infected by two different phages"
- [intro] The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra or any other high-resolution MS technique"
- [readme] MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above): "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
