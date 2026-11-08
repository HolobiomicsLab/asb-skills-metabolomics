---
name: nmr-doublet-signal-interpretation
description: Use when 1H NMR plasma spectra (e.g. FGENTCARD metabo_SE SummarizedExperiment)
  contain unassigned peaks linked to a phenotype and you need a metabolite assignment
  via driver-signal correlation.
when_to_use_negative:
- Input is already a validated metabolite table or annotated feature list — STOCSY
  adds no assignment value there.
- The driver ppm corresponds to a broad/solvent-suppressed region with no distinct
  doublet to anchor the correlation.
- Data are LC-MS or other non-NMR feature intensities where STOCSY's NMR spin-system
  logic does not apply.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3337
- http://edamontology.org/topic_0003
tools:
- name: MWASTools
  role: R/Bioconductor package supplying STOCSY_NMR() to compute the driver-anchored
    covariance/correlation pseudo-spectrum and render the STOCSY figure.
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Language environment in which MWASTools is implemented (version >=3.4) and
    where the metabo_SE SummarizedExperiment is loaded and the STOCSY analysis is
    executed.
- name: knitr
  role: Vignette engine used to render the STOCSY figure and the structured CSV record
    via the rmarkdown workflow (fig.path='figures/').
- name: rmarkdown
  role: Document format for the reproducible vignette/knitr report containing the
    STOCSY figure and CSV output.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:78a3a69d017fc7dbfd845ae7e158afb176af58979c60e3b73b5f4b1ecff0d480
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3337
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nmr-doublet-signal-interpretation

## Summary

A skill for metabolite assignment in 1H NMR metabonomic data using STOCSY (Statistical Total Correlation Spectroscopy): a driver signal (e.g. a doublet at δ 1.04 ppm) is correlated across samples to surface co-correlating peaks, letting unknown BMI-associated signals be mapped to a metabolite (here valine).

## When to use

Use when 1H NMR plasma spectra (e.g. FGENTCARD metabo_SE SummarizedExperiment) contain unassigned peaks linked to a phenotype and you need a metabolite assignment via driver-signal correlation.

## When NOT to use

- Input is already a validated metabolite table or annotated feature list — STOCSY adds no assignment value there.
- The driver ppm corresponds to a broad/solvent-suppressed region with no distinct doublet to anchor the correlation.
- Data are LC-MS or other non-NMR feature intensities where STOCSY's NMR spin-system logic does not apply.

## Inputs

- metabo_SE: a SummarizedExperiment object holding the metabonomic 1H NMR spectra (e.g. the MWASTools example FGENTCARD plasma dataset)
- ppm_query: numeric chemical shift of the driver signal (e.g. 1.04)

## Outputs

- STOCSY pseudo-spectrum plot (knitr figure, fig.path='figures/') showing driver and co-correlated peaks
- structured CSV of highest-correlated ppm signals supporting the metabolite assignment (e.g. valine at δ 1.04 and δ 0.99 doublets)

## How to apply

Load the metabonomic NMR dataset as a SummarizedExperiment (metabo_SE) in R/MWASTools, then call STOCSY_NMR with a chosen ppm driver (e.g. ppm_query=1.04) to compute a covariance/correlation pseudo-spectrum. Inspect the rendered STOCSY plot: peaks sharing high correlation with the driver (here the δ 1.04 doublet and its co-correlated δ 0.99 doublet) belong to the same metabolite multiplet pattern, identifying valine. Write the highly-correlated ppm signals into a structured CSV record so the assignment is auditable, using the knitr figure path (fig.path='figures/') for the plot. Rationale: STOCSY exploits inter-metabolite correlations within the same molecule's spin system, so co-varying NMR features across a cohort reveal metabolite identity without reference spectra.

## Related tools

- **MWASTools** (R/Bioconductor package supplying STOCSY_NMR() to compute the driver-anchored covariance/correlation pseudo-spectrum and render the STOCSY figure.) — https://bioconductor.org/packages/MWASTools/
- **R** (Language environment in which MWASTools is implemented (version >=3.4) and where the metabo_SE SummarizedExperiment is loaded and the STOCSY analysis is executed.)
- **knitr** (Vignette engine used to render the STOCSY figure and the structured CSV record via the rmarkdown workflow (fig.path='figures/').)
- **rmarkdown** (Document format for the reproducible vignette/knitr report containing the STOCSY figure and CSV output.)

## Examples

```
STOCSY_NMR(metabo_SE, ppm_query = 1.04)
```

## Evaluation signals

- The rendered STOCSY plot shows the driver doublet at δ 1.04 ppm and a highly correlated doublet at δ 0.99 ppm, matching the paper's valine assignment.
- Extracted ppm values from the STOCSY output are written to the structured CSV record and match the peaks highlighted in the figure.
- The figure is written to the knitr fig.path='figures/' directory and is reproducible from the same metabo_SE input.
- No un-correlated peaks appear in the pseudo-spectrum beyond the driver and its co-correlated doublet — confirming the correlation was anchored to the chosen ppm driver.

## Limitations

- Assignment depends on the chosen driver ppm; a wrong driver peak yields a misleading pseudo-spectrum.
- STOCSY assumes inter-metabolite correlations hold across the cohort; confounders like age and gender can distort observed associations if not handled (the paper notes univariate MWAS models adjust for these confounders).
- No reference-spectrum library is consulted — the assignment is inference from correlation patterns, so novel metabolites with no known doublet pattern may be mis-assigned.

## Evidence

- [abstract] Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of statistical outcomes; metabolite assignment using statistical total correlation spectroscopy (STOCSY): "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
- [results] With δ 1.04 as the driver signal, the STOCSY plot highlights two doublets at δ 1.04 and δ 0.99, indicating that the unknown BMI-associated signal corresponds to valine.: [evidence span withheld — anchor status: not_found]
- [methods] Call STOCSY_NMR(metabo_SE, ppm_query = 1.04) to compute the covariance/correlation pseudo-spectrum using the δ 1.04 ppm signal as the driver variable.: [evidence span withheld — anchor status: not_found]
- [intro] metabolite assignment using Statistical Total Correlation Spectroscopy (STOCSY): "metabolite assignment using Statistical Total Correlation Spectroscopy (STOCSY)"
