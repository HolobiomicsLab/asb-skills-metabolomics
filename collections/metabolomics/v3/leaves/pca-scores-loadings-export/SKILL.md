---
name: pca-scores-loadings-export
description: Use when you have SMolESY-select relative concentration values for the
  22 SMolESY-select metabolites and want to verify whether PCA of the panel recapitulates
  the biomarker signal seen when analysing the full 1H NMR profile.
when_to_use_negative:
- When the input is the full 1H NMR spectrum rather than the SMolESY-select 22-metabolite
  relative concentration table — that case calls for the whole-profile biomarker discovery
  workflow instead
- 'When only raw Bruker IVDr spectra are available: SMolESY-select assignment and
  integration must be run first to produce the 22-metabolite matrix'
- When the metabolites of interest are not among the 22 targeted spin systems covered
  by the SMolESY-select bins
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_0157
tools:
- name: SMolESY-select
  role: Produces the 22-metabolite relative concentration matrix that PCA is applied
    to; its SMolESY resolution-enhancement and baseline-suppression features are the
    upstream source of the integrated values
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Underlying spectral transformation (resolution enhancement and macromolecular
    signal suppression) whose output feeds SMolESY-select bin assignment and integration
  repo: https://github.com/pantakis/SMolESY
- name: principal component analysis (PCA)
  role: Decomposition of the 22-metabolite concentration matrix into PC scores and
    loadings for biomarker discovery
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:4f1ad71f6db1f4bae5ce37ed3ca6ebcf91d0d6c1638363c9397fc6cde05de2e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - SMolESY
  - SMolESY-select
  - principal component analysis (PCA)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pca-scores-loadings-export

## Summary

Runs PCA on the 22 SMolESY-derived metabolite relative concentration values and exports PC scores, PC loadings, and a PC1-vs-PC2 plot, so that whole-spectrum 1H NMR biomarker structure can be recovered from a compact 22-variable panel.

## When to use

Use when you have SMolESY-select relative concentration values for the 22 SMolESY-select metabolites and want to verify whether PCA of the panel recapitulates the biomarker signal seen when analysing the full 1H NMR profile.

## When NOT to use

- When the input is the full 1H NMR spectrum rather than the SMolESY-select 22-metabolite relative concentration table — that case calls for the whole-profile biomarker discovery workflow instead
- When only raw Bruker IVDr spectra are available: SMolESY-select assignment and integration must be run first to produce the 22-metabolite matrix
- When the metabolites of interest are not among the 22 targeted spin systems covered by the SMolESY-select bins

## Inputs

- SMolESY-select relative concentration matrix for the 22 SMolESY-select metabolites (rows = serum/plasma samples, columns = metabolites), from a cohort such as MTBLS395
- Cohort metadata with sample grouping/labels for coloring the scores plot

## Outputs

- PC scores table as CSV (one row per serum sample, one column per retained PC)
- PC loadings table as CSV (one row per metabolite, one column per retained PC)
- PC1-versus-PC2 scores plot as PNG, points colored by sample grouping
- Recorded PC1/PC2 explained-variance fractions accompanying the exported tables

## How to apply

Load the SMolESY-select relative concentration matrix (22 metabolite columns, one row per serum/plasma sample) produced by the preceding SMolESY-select integration step, e.g. from the MTBLS395 serum cohort. Center and scale the 22 variables, then run PCA (scikit-learn PCA or equivalent decomposition) to obtain PC scores per sample and PC loadings per metabolite. Retain the PCs needed to capture the dominant variance (record the PC1/PC2 explained-variance fractions alongside the outputs). Export the scores table (rows = samples, columns = PCs) and the loadings table (rows = metabolites, columns = PCs) as CSV files, and render a PC1-versus-PC2 scores plot colored by the sample grouping available in the cohort metadata. Rationale: the article demonstrates that PCA of the 22 SMolESY-select plasma metabolites' relative concentration values, in a cohort of 361 plasma samples from two independent NMR data sets, efficiently reveals the same biomarkers as a whole-profile biomarker discovery approach.

## Related tools

- **SMolESY-select** (Produces the 22-metabolite relative concentration matrix that PCA is applied to; its SMolESY resolution-enhancement and baseline-suppression features are the upstream source of the integrated values) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Underlying spectral transformation (resolution enhancement and macromolecular signal suppression) whose output feeds SMolESY-select bin assignment and integration) — https://github.com/pantakis/SMolESY
- **principal component analysis (PCA)** (Decomposition of the 22-metabolite concentration matrix into PC scores and loadings for biomarker discovery)

## Examples

```
python -c "import pandas as pd; from sklearn.decomposition import PCA; X = pd.read_csv('smolesy_select_22.csv').values; m = PCA(n_components=2).fit(X); pd.DataFrame(m.transform(X), columns=['PC1','PC2']).to_csv('scores.csv'); pd.DataFrame(m.components_.T, columns=['PC1','PC2']).to_csv('loadings.csv')"
```

## Evaluation signals

- Scores table row count equals the number of serum/plasma samples in the cohort (e.g. 361 in the MTBLS395 plasma example)
- Loadings table has exactly 22 rows (one per SMolESY-select metabolite)
- PC1/PC2 explained-variance fractions are recorded and reported alongside the exported CSVs
- Scores plot shows PC1 vs PC2 with points colored by the sample grouping from the cohort metadata
- PC structure of the 22-metabolite panel matches the biomarker pattern obtained from the whole 1H NMR profile for the same samples

## Limitations

- Applicable only to the 22 metabolite spin systems defined by SMolESY-select bins; metabolites outside that panel are not represented
- Downstream interpretability depends on the preceding SMolESY-select integration being correct (assignment accuracy >99.5% across the 22 targets was required in the article)
- The SMolESY-select GUI and usage video (Figure S9, Video S1) are supplementary artifacts only, so the practical invocation must be reconstructed from the text
- No software version is stated for SMolESY-select or SMolESY, so reproducing the exact upstream integration may require contacting the authors

## Evidence

- [results] PCA of the 22 SMolESY-select metabolites recapitulates whole-profile biomarkers: "principal component analysis (PCA) of the 22 plasma metabolites’ relative concentration values from SMolESY-select"
- [results] Cohort composition underpinning the PCA result: [evidence span withheld — anchor status: not_found]
- [results] Upstream SMolESY-select moving-average filtering that generates the integrated values: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data of the selected windows"
- [abstract] SMolESY-select assignment accuracy prerequisite: "yielding an assignment accuracy greater than 99.5% for all 22"
- [other] Export requirement for scores and loadings tables: [evidence span withheld — anchor status: not_found]
- [discussion] Repository availability for SMolESY-select: "it is freely available to download at https://github.com/pantakis/SMolESY-select"
