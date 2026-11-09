---
name: figure-reproduction-from-publication
description: Use when a published NMR metabonomics figure (e.g. a MWAS skyline plot
  of phenotype-metabolite associations) must be regenerated in R from the authors'
  package, its vignette SummarizedExperiment data, and documented parameter settings.
when_to_use_negative:
- The target figure derives from multivariate modeling, STOCSY, or QC-PCA analysis
  rather than the univariate MWAS skyline visualization of association results.
- The data are not 1H NMR spectra with a ppm axis (e.g. LC-MS feature tables or pre-computed
  peak lists), since the skyline plot's x-axis requires NMR chemical-shift coordinates.
- The MWAS association results do not yet exist — run the MWASTools association models
  (partial correlations, generalized linear models) first before visualizing.
edam_operation: http://edamontology.org/operation_0337
edam_topics: []
tools:
- name: MWASTools
  role: Provides MWAS_skylineNMR and the metabo_SE example data used to regenerate
    the BMI skyline figure
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Execution environment (version >= 3.4) in which the vignette and figure reproduction
    run
- name: knitr
  role: Vignette engine that executes the R chunks producing the exported figure
- name: rmarkdown
  role: Vignette rendering backend (knitr::rmarkdown) for the reproduction workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:bff131a0db8e18a9eb069bf49513d5ea67c873a8b191a20e6606c7aa81e88b14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# figure-reproduction-from-publication

## Summary

Reproduces a published metabonomics figure by re-running the authors' R package calls on their shipped example data — here rendering the BMI MWAS NMR skyline plot with MWASTools' MWAS_skylineNMR and verifying the exported PNG against Figure 1 of the Bioinformatics 2017 paper.

## When to use

Use when a published NMR metabonomics figure (e.g. a MWAS skyline plot of phenotype-metabolite associations) must be regenerated in R from the authors' package, its vignette SummarizedExperiment data, and documented parameter settings.

## When NOT to use

- The target figure derives from multivariate modeling, STOCSY, or QC-PCA analysis rather than the univariate MWAS skyline visualization of association results.
- The data are not 1H NMR spectra with a ppm axis (e.g. LC-MS feature tables or pre-computed peak lists), since the skyline plot's x-axis requires NMR chemical-shift coordinates.
- The MWAS association results do not yet exist — run the MWASTools association models (partial correlations, generalized linear models) first before visualizing.

## Inputs

- metabo_SE: MWASTools example SummarizedExperiment object with 1H NMR metabonomic data and QC reference sample metadata
- MWAS_BMI: results matrix of BMI metabolome-wide association statistics (p-values) keyed to the NMR spectral variables
- ref_sample identifier ('QC1') selecting the QC reference spectrum from the sample metadata

## Outputs

- MWAS_skyline_BMI.png: skyline plot of sign-adjusted log10 p-values vs 1H NMR chemical shift (ppm), window δ 1.60–0.80 ppm, exported under figures/
- A figure visually matching Figure 1 of the MWASTools paper

## How to apply

Load the MWASTools example SummarizedExperiment object (metabo_SE) that holds the BMI MWAS results matrix (MWAS_BMI) and its QC reference sample metadata. Call MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample="QC1") to render the skyline plot, which draws sign-adjusted log10 p-values on the y-axis against 1H NMR chemical shift in ppm on the x-axis. Restrict the displayed window to the δ 1.60–0.80 ppm lipid region so the reproduced panel matches the paper's framing. Export the figure as MWAS_skyline_BMI.png under the figures/ vignette path and visually compare it against Figure 1 of the paper; the reproduction relies on the vignette being executed through knitr::rmarkdown in R (>= 3.4).

## Related tools

- **MWASTools** (Provides MWAS_skylineNMR and the metabo_SE example data used to regenerate the BMI skyline figure) — https://bioconductor.org/packages/MWASTools/
- **R** (Execution environment (version >= 3.4) in which the vignette and figure reproduction run)
- **knitr** (Vignette engine that executes the R chunks producing the exported figure)
- **rmarkdown** (Vignette rendering backend (knitr::rmarkdown) for the reproduction workflow)

## Examples

```
MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample = "QC1")
```

## Evaluation signals

- The rendered PNG plots sign-adjusted log10 p-values on the y-axis against 1H NMR chemical shift (ppm) on the x-axis.
- The displayed window is restricted to the δ 1.60–0.80 ppm lipid region as in the paper.
- The QC reference sample 'QC1' from metabo_SE metadata is passed via ref_sample.
- The output file exists at figures/MWAS_skyline_BMI.png and is a valid image.
- Visual comparison of the PNG against Figure 1 of the MWASTools paper shows corresponding skyline peaks.

## Limitations

- Reproduction is grounded in the vignette's example data (metabo_SE/MWAS_BMI); applying it to new cohorts requires first running the full MWASTools MWAS pipeline (association models adjusted for confounders such as age and gender).
- The method is specific to 1H NMR skyline visualization; MWASTools' other functionalities (QC-PCA, STOCSY, model bootstrapping) use different functions.
- Reproducibility depends on the vignette engine (knitr::rmarkdown) and R version >= 3.4; no separate repository README was provided to cross-check parameters.

## Evidence

- [abstract] MWASTools pipeline: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [abstract] visualization functionality: "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
- [methods] vignette entry: "VignetteIndexEntry{MWASTools}"
- [methods] vignette engine: "%\VignetteEngine{knitr::rmarkdown}"
