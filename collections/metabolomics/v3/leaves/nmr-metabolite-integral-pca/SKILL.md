---
name: nmr-metabolite-integral-pca
description: Use when 1D ¹H NMR plasma/serum spectra have already been processed with
  SMolESY-select into a metabolite-integral matrix (22 metabolites × N samples), and
  the goal is exploratory biomarker discovery via dimension reduction rather than
  fitting/deconvolution-based profiling.
when_to_use_negative:
- Input is raw 1D ¹H NMR spectra (FIDs or frequency-domain data) that have not yet
  been processed by SMolESY-select into metabolite integrals — use the SMolESY-select
  integration skill first.
- The goal is targeted quantitation/absolute concentration estimation rather than
  exploratory biomarker discovery — SMolESY-select's direct integration step, not
  PCA of integrals, is the appropriate tool.
- Spectra were not acquired as standard one-dimensional ¹H NMR on a comparable platform
  (e.g. Bruker IVDr 600 MHz); the 22-metabolite bin/SMolESY Δv1/2 calibration is specific
  to that setup.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: SMolESY
  role: Upstream algorithm that supplies the resolution-enhanced, macromolecule-suppressed
    ¹H NMR spectra from which the 22 metabolite integrals are derived; prerequisite
    for the input matrix.
  repo: https://github.com/pantakis/SMolESY
- name: SMolESY-select
  role: Produces the 22-metabolite relative concentration values via direct integration
    of assigned spectral components; these integrals are the direct input to PCA.
  repo: https://github.com/pantakis/SMolESY-select
- name: principal component analysis (PCA)
  role: The dimension-reduction method applied to the 22 SMolESY-select integrals
    to obtain PC scores, loadings, and a PC1-vs-PC2 biomarker-discovery visualization.
- name: Bruker IVDr 600 MHz spectrometer
  role: Instrument on which the standard one-dimensional ¹H NMR spectra feeding the
    SMolESY-select/PCA workflow are acquired.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:5fcbda3950a374c98c72299cda25691070a3d912797dc7f23654bee8499c7d43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  - principal component analysis (PCA)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nmr-metabolite-integral-pca

## Summary

Run PCA on the SMolESY-select-derived relative concentration values of the 22 metabolites measured from 1D ¹H NMR plasma/serum spectra, to recover the same biomarker structure as a full-profile biomarker discovery approach at a fraction of the computational cost.

## When to use

Use when 1D ¹H NMR plasma/serum spectra have already been processed with SMolESY-select into a metabolite-integral matrix (22 metabolites × N samples), and the goal is exploratory biomarker discovery via dimension reduction rather than fitting/deconvolution-based profiling.

## When NOT to use

- Input is raw 1D ¹H NMR spectra (FIDs or frequency-domain data) that have not yet been processed by SMolESY-select into metabolite integrals — use the SMolESY-select integration skill first.
- The goal is targeted quantitation/absolute concentration estimation rather than exploratory biomarker discovery — SMolESY-select's direct integration step, not PCA of integrals, is the appropriate tool.
- Spectra were not acquired as standard one-dimensional ¹H NMR on a comparable platform (e.g. Bruker IVDr 600 MHz); the 22-metabolite bin/SMolESY Δv1/2 calibration is specific to that setup.

## Inputs

- SMolESY-select output: matrix of relative concentration values (integrals) for the 22 metabolites, one row per 1D ¹H NMR plasma/serum sample (e.g. from the MTBLS395 serum cohort, 361 samples in the article's demonstration)
- Sample metadata/grouping labels for coloring the PCA scores plot (e.g. MTBLS395 serum cohort annotations)

## Outputs

- PCA PC scores table (one row per sample, columns for each retained PC) exported as CSV
- PCA PC loadings table (one row per metabolite, columns for each retained PC) exported as CSV
- PC1-vs-PC2 scores plot colored by sample grouping, saved as PNG
- Recorded PC1/PC2 variance fractions alongside the exported tables

## How to apply

Start from the SMolESY-select output: the 22 metabolite relative concentration values (integrals) computed from standard one-dimensional ¹H NMR spectra acquired on a Bruker IVDr 600 MHz instrument. Center and scale these 22 variables (standard pre-processing for PCA on concentration data) and run PCA, e.g. via scikit-learn's PCA, on a cohort such as the 361-sample MTBLS395 serum set. The article demonstrates this exact procedure on a cohort of 361 plasma samples composed of two independent NMR data sets, showing that PCA of only the 22 SMolESY-select integrals reveals the same biomarkers as a whole-1H-NMR-profile analysis. Export PC scores (one row per sample) and PC loadings (one row per metabolite), record the PC1/PC2 variance fractions, and generate a PC1-vs-PC2 scores plot colored by sample grouping from the study metadata (e.g. MTBLS395 serum cohort annotations). Correct application is judged by verifying row counts (N samples for scores; 22 rows for loadings), sane variance fractions on PC1/PC2, and visually inspecting the scores plot for grouping consistent with the known sample classes.

## Related tools

- **SMolESY** (Upstream algorithm that supplies the resolution-enhanced, macromolecule-suppressed ¹H NMR spectra from which the 22 metabolite integrals are derived; prerequisite for the input matrix.) — https://github.com/pantakis/SMolESY
- **SMolESY-select** (Produces the 22-metabolite relative concentration values via direct integration of assigned spectral components; these integrals are the direct input to PCA.) — https://github.com/pantakis/SMolESY-select
- **principal component analysis (PCA)** (The dimension-reduction method applied to the 22 SMolESY-select integrals to obtain PC scores, loadings, and a PC1-vs-PC2 biomarker-discovery visualization.)
- **Bruker IVDr 600 MHz spectrometer** (Instrument on which the standard one-dimensional ¹H NMR spectra feeding the SMolESY-select/PCA workflow are acquired.)

## Examples

```
from sklearn.decomposition import PCA; import pandas as pd; X = pd.read_csv('smolesy_select_integrals.csv', index_col=0); Z = (X - X.mean()) / X.std(); model = PCA(n_components=5).fit(Z); scores = pd.DataFrame(model.transform(Z), index=X.index); scores.to_csv('pc_scores.csv'); pd.DataFrame(model.components_.T, index=X.columns).to_csv('pc_loadings.csv')
```

## Evaluation signals

- Scores table row count equals the number of input samples (e.g. 361 for the MTBLS395 serum cohort); loadings table row count equals 22 (one per metabolite).
- PC1 and PC2 variance fractions are recorded and are sane (each between 0 and 1, sum of all retained PCs ≤ 1).
- PC1-vs-PC2 scores plot shows grouping consistent with the sample metadata (e.g. MTBLS395 cohort annotations) and no degenerate single-cluster collapse.
- All 22 metabolite variables are centered and scaled before decomposition (verify no unnormalized raw integrals leak into the loadings).
- Loadings magnitude/ordering should reproduce the biomarker structure previously identified by full-profile analysis on the same cohort, per the article's validation.

## Limitations

- PCA of the 22 SMolESY-select integrals is a targeted-panel approach; it does not recover structure from metabolites outside the predefined 22-metabolite panel (e.g. signals outside the calibrated SMolESY bins).
- The article's demonstration relies on 1D ¹H NMR spectra acquired with Bruker IVDr 600 MHz instrumentation; applying this to other field strengths or acquisition protocols may require recalibration of SMolESY Δv1/2 bins.
- The >8800-sample validation set used to benchmark SMolESY-select's assignment accuracy (>99.5% for all 22 metabolites) is not named or given a repository accession, limiting independent re-verification of the upstream quantitation step.
- GUI and usage documentation for SMolESY-select exist only as supplementary artifacts (Figure S9, Video S1) not included in the main text, so scripting the upstream integration step may require consulting the SI.

## Evidence

- [results] principal component analysis (PCA) of the 22 plasma metabolites' relative concentration values: [evidence span withheld — anchor status: not_found]
- [results] MTBLS395 serum cohort: "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
- [results] SMolESY-select direct integration: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [results] SMolESY-select moving average filter: "SMolESY-select utilizes a moving average filter spanning 11-points"
- [abstract] assignment accuracy: "yielding an assignment accuracy greater than 99.5% for all 22 metabolites"
- [methods] instrument platform: "NMR samples were acquired using Bruker IVDr 600 MHz spectrometers"
