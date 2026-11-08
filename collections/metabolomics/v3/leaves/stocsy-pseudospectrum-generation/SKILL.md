---
name: stocsy-pseudospectrum-generation
description: Use when an MWAS on metabonomic 1H NMR spectra (e.g. plasma spectra vs
  BMI) flags unassigned ppm signals and you need to identify the underlying metabolite
  by correlating all spectral variables against a driver ppm.
when_to_use_negative:
- The input is not a matrix/vector of spectral intensity variables across samples
  (e.g. a feature table of MS peak areas), since STOCSY relies on dense ppm-resolution
  NMR variables
- The metabolite-phenotype association step (MWAS) has not yet identified the ppm
  signals of interest — STOCSY assigns already-flagged features rather than detecting
  associations
- Only a single spectrum is available; STOCSY requires across-sample statistical correlation
  between spectral variables
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: MWASTools
  role: Provides STOCSY_NMR() to compute and plot the covariance/correlation pseudo-spectrum
    from the SummarizedExperiment NMR dataset
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment (version >= 3.4) in which the STOCSY analysis is executed
- name: knitr
  role: Vignette/report engine that renders the STOCSY figure to fig.path='figures/'
- name: rmarkdown
  role: Vignette engine used to produce the reproducible analysis document containing
    the STOCSY workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:090c034ca0f38f1de69012a128e09a92fdc5a90a0bb0980839566bc4721bfcd1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# stocsy-pseudospectrum-generation

## Summary

Generates a STOCSY (Statistical Total Correlation Spectroscopy) pseudo-spectrum from a cohort of 1H NMR spectra to assign metabolite identity to statistically associated spectral features, by computing covariance/correlation of every ppm variable against a chosen driver signal.

## When to use

Use when an MWAS on metabonomic 1H NMR spectra (e.g. plasma spectra vs BMI) flags unassigned ppm signals and you need to identify the underlying metabolite by correlating all spectral variables against a driver ppm.

## When NOT to use

- The input is not a matrix/vector of spectral intensity variables across samples (e.g. a feature table of MS peak areas), since STOCSY relies on dense ppm-resolution NMR variables
- The metabolite-phenotype association step (MWAS) has not yet identified the ppm signals of interest — STOCSY assigns already-flagged features rather than detecting associations
- Only a single spectrum is available; STOCSY requires across-sample statistical correlation between spectral variables

## Inputs

- SummarizedExperiment object of metabonomic 1H NMR spectra across a cohort (e.g. MWASTools example dataset metabo_SE)
- ppm_query: ppm value of the driver spectral signal (e.g. 1.04)

## Outputs

- STOCSY pseudo-spectrum of covariance/correlation values across ppm variables
- STOCSY plot highlighting the driver signal and correlated peaks (written to fig.path='figures/')
- Structured CSV record of highest-correlated ppm signals supporting the metabolite assignment

## How to apply

Load the metabonomic NMR dataset as a SummarizedExperiment object (e.g. the MWASTools example metabo_SE) in R. Call STOCSY_NMR() with ppm_query set to the ppm of the driver signal — e.g. 1.04 for the BMI-associated unknown doublet — to compute the covariance/correlation pseudo-spectrum across all ppm variables. Render the STOCSY plot to the knitr figure path (fig.path='figures/') and read off the peaks most highly correlated with the driver. A driver doublet at δ 1.04 ppm accompanied by a highly correlated doublet at δ 0.99 ppm supports assigning the signal pair to the valine methyl groups. Export the top-correlated ppm values to a structured CSV record to document the assignment.

## Related tools

- **MWASTools** (Provides STOCSY_NMR() to compute and plot the covariance/correlation pseudo-spectrum from the SummarizedExperiment NMR dataset) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment (version >= 3.4) in which the STOCSY analysis is executed)
- **knitr** (Vignette/report engine that renders the STOCSY figure to fig.path='figures/')
- **rmarkdown** (Vignette engine used to produce the reproducible analysis document containing the STOCSY workflow)

## Examples

```
STOCSY_NMR(metabo_SE, ppm_query = 1.04)
```

## Evaluation signals

- The pseudo-spectrum shows the driver doublet at the queried ppm (δ 1.04 ppm) with maximum correlation (self-correlation ~1)
- A second highly correlated doublet appears at δ 0.99 ppm, matching the coupled valine methyl-group pattern
- The STOCSY figure is written to the knitr figure path (fig.path='figures/') and displays both highlighted doublets
- The exported CSV contains the highest-correlated ppm signals consistent with the valine assignment (δ 1.04 and δ 0.99)
- The assignment agrees with known valine chemical shifts reported for plasma 1H NMR spectra

## Limitations

- STOCSY assigns metabolites by correlation patterns only; overlapping signals from co-correlated metabolites can confound assignment
- The pseudo-spectrum depends on the choice of driver ppm; a wrong driver signal yields misleading correlation patterns
- Requires cohort-scale NMR data with variables aligned on a common ppm grid (SummarizedExperiment format in MWASTools)

## Evidence

- [abstract] STOCSY metabolite assignment: "metabolite assignment using statistical total correlation spectroscopy (STOCSY)"
- [intro] Package functionality: "metabolite assignment using Statistical Total Correlation Spectroscopy (STOCSY)"
- [abstract] Runtime requirement: "The MWASTools R package is implemented in R (version > =3.4)"
- [methods] Vignette engine: "%\VignetteEngine{knitr::rmarkdown}"
- [other] STOCSY_NMR invocation on task card: [evidence span withheld — anchor status: not_found]
- [other] Validation of valine assignment: [evidence span withheld — anchor status: not_found]
