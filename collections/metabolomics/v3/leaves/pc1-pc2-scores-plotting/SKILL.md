---
name: pc1-pc2-scores-plotting
description: Use when a matrix of relative concentration values for the 22 SMolESY-select
  metabolites (e.g. serum samples from MTBLS395) is available and you need PCA-based
  visualization/ biomarker-pattern comparison against a whole-1H NMR-profile approach.
when_to_use_negative:
- Input is already a processed PCA result (scores/loadings) rather than the raw SMolESY-select
  relative concentration matrix
- The goal is metabolite quantitation or assignment from raw 1H NMR spectra — use
  SMolESY-select integration instead, then apply this skill downstream on its 22-metabolite
  output
- Only one or a handful of metabolites are of interest without needing a multivariate
  pattern comparison against the whole NMR profile
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: principal component analysis (PCA)
  role: Dimensionality reduction of the 22 SMolESY-select metabolite relative concentration
    values to obtain PC scores and loadings for biomarker-pattern visualization
- name: SMolESY-select
  role: Upstream tool that produces the 22-metabolite relative concentration values
    fed into the PCA
  repo: https://github.com/pantakis/SMolESY-select
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:dba13423fa7f82add432d59349e9bd241e0a8c50608847611694593d57393a58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - SMolESY-select
  - principal component analysis (PCA)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pc1-pc2-scores-plotting

## Summary

Generate and validate PC1-versus-PC2 scores plots from principal component analysis (PCA) of the 22 SMolESY-select metabolite relative concentration values, demonstrating that a 22-metabolite panel reproduces biomarker patterns otherwise obtained from the whole 1H NMR profile.

## When to use

Use when a matrix of relative concentration values for the 22 SMolESY-select metabolites (e.g. serum samples from MTBLS395) is available and you need PCA-based visualization/ biomarker-pattern comparison against a whole-1H NMR-profile approach.

## When NOT to use

- Input is already a processed PCA result (scores/loadings) rather than the raw SMolESY-select relative concentration matrix
- The goal is metabolite quantitation or assignment from raw 1H NMR spectra — use SMolESY-select integration instead, then apply this skill downstream on its 22-metabolite output
- Only one or a handful of metabolites are of interest without needing a multivariate pattern comparison against the whole NMR profile

## Inputs

- SMolESY-select relative concentration matrix for the 22 metabolites (one row per sample, e.g. from the MTBLS395 serum cohort)
- MTBLS395 sample metadata providing grouping information for coloring the scores plot

## Outputs

- PC scores table (one row per serum sample, columns for each retained PC) exported as CSV
- PC loadings table (one row per metabolite, columns for each retained PC) exported as CSV
- PC1-versus-PC2 scores plot saved as a PNG figure
- Recorded PC1/PC2 variance fractions

## How to apply

Load the SMolESY-select relative concentration matrix (22 metabolite columns, one row per serum/plasma sample), center and scale the variables, and run PCA (e.g. scikit-learn's PCA) to obtain PC scores and loadings. Retain the first PCs and record the PC1/PC2 variance fractions alongside the tables. Export the PC scores table (one row per sample) and the PC loadings table (one row per metabolite) as CSV files. Generate a PC1-versus-PC2 scores plot, coloring points using the available sample grouping in the MTBLS395 metadata, and save as a PNG. Validate that the scores and loadings tables have the expected row counts (number of serum samples and 22 metabolites, respectively) and that the variance fractions are recorded; then compare the resulting biomarker pattern with that derived from the whole 1H NMR profile to confirm the 22-metabolite panel reveals the same biomarkers.

## Related tools

- **principal component analysis (PCA)** (Dimensionality reduction of the 22 SMolESY-select metabolite relative concentration values to obtain PC scores and loadings for biomarker-pattern visualization)
- **SMolESY-select** (Upstream tool that produces the 22-metabolite relative concentration values fed into the PCA) — https://github.com/pantakis/SMolESY-select

## Examples

```
from sklearn.decomposition import PCA; import pandas as pd; X = pd.read_csv('smolesy_select_22met_relconc.csv'); scores = PCA(n_components=2).fit_transform(X)
```

## Evaluation signals

- The PC scores table has one row per serum sample and the PC loadings table has exactly 22 rows (one per metabolite)
- PC1/PC2 variance fractions are recorded alongside the exported tables
- The PC1-versus-PC2 scores plot is saved as a PNG and points are colored by the sample grouping available in the MTBLS395 metadata
- Biomarkers revealed by the 22-metabolite-panel PCA match those obtained when PCA is performed on the whole 1H NMR profile

## Limitations

- The validation of the biomarker agreement used a cohort of 361 plasma samples from two independent NMR data sets; performance on other cohorts or sample types is not demonstrated here
- PCA is performed only on the 22 SMolESY-select relative concentration values — information outside this panel is not considered, so it cannot capture biomarkers outside the 22 metabolites

## Evidence

- [results] PCA of the 22 metabolites' relative concentration values from SMolESY-select, in a cohort of 361 plasma samples consisting of two independent NMR data sets, efficiently reveals the same biomarkers.: [evidence span withheld — anchor status: not_found]
- [results] principal component analysis (PCA) of the 22 plasma metabolites’ relative concentration values: "principal component analysis (PCA) of the 22 plasma metabolites’ relative concentration values"
- [results] one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395): "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
- [discussion] SMolESY-select, which demonstrates exclusive advantages: "we introduced a unique algorithm, SMolESY-select, which demonstrates exclusive advantages"
