---
name: nmr-metabolite-assignment
description: Use when 1H-NMR spectra (e.g. plasma metabonomic profiles) yield unassigned
  peaks linked to a phenotype and you need metabolite-level assignment via co-correlation
  of ppm signals.
when_to_use_negative:
- The input is already an assigned metabolite feature table — STOCSY is for de novo
  assignment of unexplained resonances.
- The spectra are LC-MS peak lists or MS/MS fragments; STOCSY relies on NMR chemical-shift
  co-variation.
- Only a single spectrum is available — correlation across many samples is required.
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: MWASTools
  role: Provides the STOCSY_NMR function that computes the covariance/correlation
    pseudo-spectrum for metabolite assignment from 1H-NMR spectra stored as SummarizedExperiment
    objects.
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment (>=3.4) in which MWASTools and the knitr/rmarkdown vignette
    rendering of the STOCSY plot are executed.
- name: knitr
  role: Vignette engine used to render the STOCSY figure into the knitr figure path
    (fig.path='figures/').
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:fc3ae8557b096303de68a2583536c1f9c8e1c036d27606e44b452cfa9cc3f883
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  - knitr
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# nmr-metabolite-assignment

## Summary

Assigns resonances in 1H-NMR metabonomic spectra to metabolites using Statistical Total Correlation Spectroscopy (STOCSY): a highly correlated ppm signal is chosen as driver and the covariance/correlation pseudo-spectrum reveals co-varying peaks of the same molecule.

## When to use

Use when 1H-NMR spectra (e.g. plasma metabonomic profiles) yield unassigned peaks linked to a phenotype and you need metabolite-level assignment via co-correlation of ppm signals.

## When NOT to use

- The input is already an assigned metabolite feature table — STOCSY is for de novo assignment of unexplained resonances.
- The spectra are LC-MS peak lists or MS/MS fragments; STOCSY relies on NMR chemical-shift co-variation.
- Only a single spectrum is available — correlation across many samples is required.

## Inputs

- 1H-NMR metabonomic spectra (SummarizedExperiment object, e.g. MWASTools example metabo_SE)
- ppm chemical shift of the unassigned driver signal (e.g. 1.04 ppm)

## Outputs

- STOCSY covariance/correlation pseudo-spectrum plot highlighting driver and correlated peaks
- CSV of highest-correlated ppm signals supporting the metabolite assignment

## How to apply

Load the 1H-NMR dataset into R as a SummarizedExperiment object (e.g. MWASTools' example metabo_SE, plasma spectra from the FGENTCARD study). Call STOCSY_NMR() with the ppm of the unassigned signal as the driver (ppm_query, e.g. 1.04). The function computes a covariance/correlation pseudo-spectrum across all samples: peaks that co-vary with the driver — such as a second methyl doublet — are highlighted, and the pattern of correlated chemical shifts identifies the metabolite (e.g. doublets at δ 1.04 and 0.99 ppm assign the signal to valine). The rationale is that peaks belonging to the same metabolite share a correlation structure across samples, so the driver's pseudo-spectrum exposes the full spin-system fingerprint. Render the STOCSY plot (knitr/rmarkdown) with the driver and correlated peaks marked, and export the top-correlated ppm values as a structured CSV for downstream annotation.

## Related tools

- **MWASTools** (Provides the STOCSY_NMR function that computes the covariance/correlation pseudo-spectrum for metabolite assignment from 1H-NMR spectra stored as SummarizedExperiment objects.) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment (>=3.4) in which MWASTools and the knitr/rmarkdown vignette rendering of the STOCSY plot are executed.)
- **knitr** (Vignette engine used to render the STOCSY figure into the knitr figure path (fig.path='figures/').)

## Examples

```
R> library(MWASTools); STOCSY_NMR(metabo_SE, ppm_query = 1.04)
```

## Evaluation signals

- The pseudo-spectrum shows the driver doublet at the query ppm (e.g. δ 1.04 ppm) together with a highly correlated doublet at δ 0.99 ppm, matching the known valine methyl-group pattern.
- The highest-correlated ppm values extracted from the STOCSY output agree with the metabolite's expected chemical shifts in published NMR reference spectra.
- The rendered STOCSY plot marks both the driver signal and the correlated partner peaks, and is written to the knitr figure path (fig.path='figures/').
- The exported CSV contains the driver ppm, correlated ppm values, and their correlation/covariance scores in a structured schema.
- The assignment is consistent with the paper's reported valine assignment for the unknown BMI-associated signal in FGENTCARD plasma spectra.

## Limitations

- STOCSY assignments are inferential: co-correlation of peaks indicates a shared metabolite but does not prove identity, so overlapping resonances from co-eluting/co-varying metabolites can be misassigned.
- The driver signal must be chosen judiciously — a driver ppm contaminated by noise or by a neighbouring metabolite's peak will produce a misleading pseudo-spectrum.
- Requires a cohort of many samples; the correlation structure cannot be estimated reliably from a single or few spectra.
- The package assumes R >= 3.4 and Bioconductor installation; older R versions are not supported.

## Evidence

- [abstract] Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of statistical outcomes; and metabolite assignment using statistical total correlation spectroscopy (STOCSY): "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
- [results] With δ 1.04 as the driver signal, the STOCSY plot highlights two doublets at δ 1.04 and δ 0.99, indicating that the unknown BMI-associated signal corresponds to valine.: [evidence span withheld — anchor status: not_found]
- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [methods] Call STOCSY_NMR(metabo_SE, ppm_query = 1.04) to compute the covariance/correlation pseudo-spectrum using the δ 1.04 ppm signal as the driver variable.: [evidence span withheld — anchor status: not_found]
