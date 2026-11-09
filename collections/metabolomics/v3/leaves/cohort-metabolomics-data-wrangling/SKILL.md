---
name: cohort-metabolomics-data-wrangling
description: Use when you have hundreds to thousands of 1D 1H NMR serum or plasma
  spectra (e.g. Bruker IVDr 600 MHz, MTBLS395) and need reliable, automated integration
  of a 22-metabolite panel followed by PCA-based biomarker discovery.
when_to_use_negative:
- Input is already a summarized feature table not derived from SMolESY-select's 22-metabolite
  panel — the PCA biomarker interpretation is specific to those relative concentration
  values.
- Only a handful of spectra are available (< the >8800-sample validation scale); the
  algorithm's >99.5% assignment accuracy was established on large cohorts and may
  not hold on small ones.
- The goal is quantitation of metabolites outside the 22-metabolite panel (e.g. lipids
  or protein signals suppressed by macromolecular suppression) — SMolESY-select's
  bins and moving-average filter were defined for those 22 spin systems.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: SMolESY
  role: Upstream computational step that performs spectral resolution enhancement
    and macromolecular signal suppression on the raw 1D 1H NMR spectra before integration
  repo: https://github.com/pantakis/SMolESY
- name: SMolESY-select
  role: Assigns and integrates the 22-metabolite panel via predefined spectral windows
    (bins) and an 11-point moving average filter, producing the relative concentration
    matrix used for PCA
  repo: https://github.com/pantakis/SMolESY-select
- name: principal component analysis (PCA)
  role: Downstream multivariate analysis applied to the 22 SMolESY-select relative
    concentration values to reveal biomarkers; scikit-learn's PCA is a suitable implementation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:867eb6d9d0d55906ad70f315485188e201675942075ea295ea461fce9eb199b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY
  tools:
  - SMolESY
  - SMolESY-select
  - principal component analysis (PCA)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cohort-metabolomics-data-wrangling

## Summary

A cohort-scale NMR metabolomics data wrangling skill: run SMolESY/SMolESY-select over large serum/plasma 1D 1H NMR cohorts, extract and assign 22 metabolite panel integrals, and feed the resulting relative concentration matrix into multivariate analyses such as PCA to reveal biomarkers.

## When to use

Use when you have hundreds to thousands of 1D 1H NMR serum or plasma spectra (e.g. Bruker IVDr 600 MHz, MTBLS395) and need reliable, automated integration of a 22-metabolite panel followed by PCA-based biomarker discovery.

## When NOT to use

- Input is already a summarized feature table not derived from SMolESY-select's 22-metabolite panel — the PCA biomarker interpretation is specific to those relative concentration values.
- Only a handful of spectra are available (< the >8800-sample validation scale); the algorithm's >99.5% assignment accuracy was established on large cohorts and may not hold on small ones.
- The goal is quantitation of metabolites outside the 22-metabolite panel (e.g. lipids or protein signals suppressed by macromolecular suppression) — SMolESY-select's bins and moving-average filter were defined for those 22 spin systems.

## Inputs

- Relative concentration matrix (samples × 22 metabolites) produced by SMolESY-select integration, e.g. from the MTBLS395 serum cohort
- Cohort metadata (e.g. MTBLS395 sample grouping labels) for coloring PCA scores
- Raw 1D 1H NMR spectra acquired on Bruker IVDr 600 MHz instruments (optional upstream input for running SMolESY/SMolESY-select)

## Outputs

- PC scores table (CSV; one row per serum sample, one column per retained PC)
- PC loadings table (CSV; one row per metabolite, one column per retained PC)
- PC1-vs-PC2 scores plot (PNG), colored by MTBLS395 sample grouping
- Recorded PC1/PC2 variance fractions alongside the exported tables

## How to apply

Load the SMolESY-select relative concentration matrix (one row per sample, one column per metabolite) for the 22 SMolESY-select metabolites, e.g. from the MTBLS395 serum cohort output of a preceding integration task. Center and scale the 22 metabolite variables and run PCA (e.g. via scikit-learn's PCA) to obtain PC scores and loadings. Export the PC scores table (rows = serum samples, columns = retained PCs) and the PC loadings table (rows = 22 metabolites, columns = retained PCs) as CSV files. Generate a PC1-versus-PC2 scores plot, coloring points according to the sample grouping available in the MTBLS395 metadata, and save it as a PNG. Validate that the scores table has as many rows as serum samples, that the loadings table has 22 rows, and record the PC1/PC2 variance fractions alongside the exported tables. Rationale: the algorithm was developed and validated on >4000 training and >8800 test serum/plasma spectra, achieving >99.5% assignment accuracy for all 22 metabolites, so its downstream PCA on relative concentration values is the intended route to biomarker discovery.

## Related tools

- **SMolESY** (Upstream computational step that performs spectral resolution enhancement and macromolecular signal suppression on the raw 1D 1H NMR spectra before integration) — https://github.com/pantakis/SMolESY
- **SMolESY-select** (Assigns and integrates the 22-metabolite panel via predefined spectral windows (bins) and an 11-point moving average filter, producing the relative concentration matrix used for PCA) — https://github.com/pantakis/SMolESY-select
- **principal component analysis (PCA)** (Downstream multivariate analysis applied to the 22 SMolESY-select relative concentration values to reveal biomarkers; scikit-learn's PCA is a suitable implementation)

## Examples

```
python -m smolesy_select.run --input MTBLS395_spectra --output relative_concentrations.csv; python -c "import pandas as pd; from sklearn.decomposition import PCA; X = pd.read_csv('relative_concentrations.csv', index_col=0); Xs = (X - X.mean()) / X.std(); pca = PCA(); scores = pca.fit_transform(Xs); scores.to_csv('pc_scores.csv'); pd.DataFrame(pca.components_, columns=X.columns).to_csv('pc_loadings.csv')"
```

## Evaluation signals

- Scores table row count equals the number of serum samples; loadings table has exactly 22 rows (one per metabolite).
- PC1 and PC2 variance fractions are recorded alongside the exported CSV tables.
- The PC1-versus-PC2 scores plot is a PNG colored by the MTBLS395 sample grouping from the metadata.
- Downstream PCA on the SMolESY-select relative concentration values should reveal the same biomarkers as a whole-1H NMR profile biomarker discovery approach.
- Upstream assignment accuracy for all 22 metabolites should be greater than 99.5%, as reported for the SMolESY-select algorithm on the >8800 validation spectra.

## Limitations

- The algorithm's performance was established on >4000 training and >8800 test serum/plasma spectra; its >99.5% assignment accuracy may not generalize to very small or unusual cohorts.
- Only standard one-dimensional 1H NMR spectra are used — 2D experiments, complex deconvolution, or spectral baseline fitting are deliberately avoided, so metabolites with heavily overlapping signals may be harder to resolve.
- SMolESY-select and SMolESY versions are not stated in the article; the GUI (Figure S9) and usage video (Video S1) exist only as supplementary artifacts.
- The competing fitting/deconvolution approaches are cited only qualitatively (several minutes of computation per spectral set) without naming specific software, making direct timing comparisons difficult.

## Evidence

- [results] PCA of the 22 metabolites' relative concentration values from SMolESY-select: "principal component analysis (PCA) of the 22 plasma metabolites’ relative concentration values"
- [abstract] >4000 training and >8800 test serum/plasma spectra: "developed using >4000 NMR serum and plasma spectra, was evaluated using an additional >8800 spectra"
- [results] SMolESY-select assigns and integrates the 22-metabolite panel using predefined spectral windows: "the algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins)"
- [results] 11-point moving average filter applied within selected windows: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data"
- [abstract] MACROMOLECULAR suppression and spectral resolution enhancement as upstream rationale: "bolstered by spectral resolution enhancement and macromolecular signal suppression"
- [results] Downstream PCA efficiently reveals the same biomarkers as a whole-1H NMR profile approach: [evidence span withheld — anchor status: not_found]
- [abstract] Only standard 1D 1H NMR spectra are needed — no complex deconvolution or baseline fitting: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra"
