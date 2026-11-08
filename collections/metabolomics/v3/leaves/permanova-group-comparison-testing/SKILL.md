---
name: permanova-group-comparison-testing
description: Use when FT-ICR MS peak-intensity data with assigned molecular formulas
  have been normalized and grouped by treatment metadata, and you need a significance
  test of between-group differences in metabolite composition that complements ordination.
when_to_use_negative:
- Raw spectra whose molecular formula assignment and peak filtering (m/z, 13C isotopes,
  0.5 ppm error, minimum sample count) have not yet been done — MetaboDirect requires
  pre-processed formula-assigned data
- Situations where a single significant PERMANOVA result will be over-interpreted
  as visible group separation — the paper shows significant p-values can occur when
  NMDS/PCA ordination fails to discriminate groups
- Data lacking sample grouping metadata, since PERMANOVA tests dissimilarity against
  a grouping factor
edam_operation: http://edamontology.org/operation_3797
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Pipeline whose statistical analysis step runs PERMANOVA on FT-ICR MS metabolomic
    profiles and exports statistics as .csv
  repo: https://github.com/Coayala/MetaboDirect
- name: vegan
  role: R library used by MetaboDirect for PERMANOVA/dissimilarity-based group testing
- name: pandas
  role: Loads and holds the normalized peak-intensity DataFrame used to build dissimilarity
    matrices
- name: scikit-learn
  role: Supports the multivariate statistical analysis (PCA) run alongside PERMANOVA
    in the MetaboDirect stack
- name: pmartR
  role: R package used in MetaboDirect's normalization tests feeding the normalized
    data that PERMANOVA consumes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:cbfec1d1eb9d7ce1b1809722ac81824e97574ac5b5cfccb4e45f0a9bcee6640f
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
  - pmartR
  - scikit-learn
  - seaborn
  - vegan
  edam_operation: http://edamontology.org/operation_3797
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# permanova-group-comparison-testing

## Summary

Non-parametric PERMANOVA testing of whether sample groups (e.g., different infecting phages in a bacterium-phage FT-ICR MS metabolomics experiment) differ significantly in metabolite content, applied within MetaboDirect's statistical analysis step on normalized peak-intensity tables and dissimilarity matrices.

## When to use

Use when FT-ICR MS peak-intensity data with assigned molecular formulas have been normalized and grouped by treatment metadata, and you need a significance test of between-group differences in metabolite composition that complements ordination.

## When NOT to use

- Raw spectra whose molecular formula assignment and peak filtering (m/z, 13C isotopes, 0.5 ppm error, minimum sample count) have not yet been done — MetaboDirect requires pre-processed formula-assigned data
- Situations where a single significant PERMANOVA result will be over-interpreted as visible group separation — the paper shows significant p-values can occur when NMDS/PCA ordination fails to discriminate groups
- Data lacking sample grouping metadata, since PERMANOVA tests dissimilarity against a grouping factor

## Inputs

- Normalized peak-abundance table with assigned molecular formulas (e.g., .csv peak-intensity matrix from FT-ICR MS processing)
- Sample grouping metadata (e.g., which phage infected each bacterial culture)
- Dissimilarity matrices computed from normalized intensities (Bray-Curtis, Euclidean, Jaccard) and molecular class composition vectors

## Outputs

- PERMANOVA statistics table exported as a .csv file (test statistics and p-values per grouping factor)
- Interpretation of group significance (e.g., phage type significantly affects metabolite content, p < 0.05)

## How to apply

Load the normalized peak-intensity table (peaks with assigned molecular formulas) into a pandas DataFrame, then compute pairwise sample dissimilarity matrices using Bray-Curtis, Euclidean, and Jaccard metrics on the normalized intensities (and optionally molecular class composition vectors). Run PERMANOVA on each dissimilarity matrix against the sample grouping metadata — in MetaboDirect this is executed with the R package vegan (part of the pipeline's required R libraries) — and export the PERMANOVA statistics table as a .csv file. Interpret significance against the grouping factor, but always alongside ordination: in the paper's bacterium-phage data set, PERMANOVA detected a significant effect of infecting phage type on metabolite content (p < 0.05) even though NMDS/PCA could not visually discriminate the groups, indicating the differences were subtle. Run the test_normalization (SPANS) companion script first so the PERMANOVA input reflects the best normalization method.

## Related tools

- **MetaboDirect** (Pipeline whose statistical analysis step runs PERMANOVA on FT-ICR MS metabolomic profiles and exports statistics as .csv) — https://github.com/Coayala/MetaboDirect
- **vegan** (R library used by MetaboDirect for PERMANOVA/dissimilarity-based group testing)
- **pandas** (Loads and holds the normalized peak-intensity DataFrame used to build dissimilarity matrices)
- **scikit-learn** (Supports the multivariate statistical analysis (PCA) run alongside PERMANOVA in the MetaboDirect stack)
- **pmartR** (R package used in MetaboDirect's normalization tests feeding the normalized data that PERMANOVA consumes)

## Examples

```
Rscript -e "library(vegan); adonis2(bray_curtis_dist ~ phage_treatment, data=meta, permutations=999)"
```

## Evaluation signals

- PERMANOVA statistics table exists as a .csv with test statistics and p-values for each dissimilarity metric (Bray-Curtis, Euclidean, Jaccard)
- p-value for the phage-type grouping factor is < 0.05 in the bacterium-phage data set, matching the published finding
- Significance is reported together with ordination results; agreement between 'significant but ordination-indistinguishable' is the expected pattern for subtle differences
- Input intensities were normalized using the method chosen by the test_normalization (SPANS) companion script, not raw peak abundances
- Isotopic-carbon-assigned masses (~200 removed in the bacterium-phage set) are excluded from the matrix before testing

## Limitations

- DI-MS direct injection cannot separate chemical isomers and suffers signal suppression or enhancement that can confound downstream data analysis
- Significant PERMANOVA results do not guarantee that ordination (NMDS/PCA) will visually separate groups — differences can be statistically significant yet subtle
- MetaboDirect provides a quick overview of the data upon which more in-depth analysis must be built

## Evidence

- [results] PERMANOVA finding: [evidence span withheld — anchor status: not_found]
- [results] ordination contrast: [evidence span withheld — anchor status: not_found]
- [methods] workflow dissimilarity step: [evidence span withheld — anchor status: not_found]
- [intro] statistical analysis scope: "MetaboDirect was designed to facilitate data exploration, data visualization, chemodiversity, and statistical analysis of metabolomic profiles"
- [readme] vegan dependency: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above)"
