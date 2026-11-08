---
name: nmr-chemical-shift-axis-plotting
description: Use when MWAS results from 1H NMR metabonomic data need to be visualized
  as a skyline plot against chemical shift (ppm), e.g. reproducing paper figures like
  the BMI association plot over the δ 0.80–1.60 ppm lipid region.
when_to_use_negative:
- Input data are mass spectrometry or other non-spectral metabolomics results — the
  skyline is indexed by 1H NMR chemical shift (ppm), not m/z or retention time
- The analysis goal is statistical association modeling itself (partial correlations
  / generalized linear models with confounder adjustment) rather than visualization
  of already-computed MWAS results
- A tabular or Manhattan-style summary of associations is needed rather than a spectrum-anchored
  skyline over a specific ppm region
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MWASTools
  role: Provides MWAS_skylineNMR, which renders the NMR skyline plot from the SummarizedExperiment
    spectra and the MWAS results matrix using a QC reference sample
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment for MWASTools (requires R >= 3.4) in which the skyline
    plotting call is executed
- name: rmarkdown
  role: Vignette engine used to render the workflow, including export of the skyline
    figure (MWAS_skyline_BMI.png) under figures/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:90e61ffab031f80c6adfbbc4b5529fde1cd2d37b544157e5ceb8f0835a603165
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nmr-chemical-shift-axis-plotting

## Summary

Renders metabolome-wide association (MWAS) results along the 1H NMR chemical shift axis as a skyline plot, using MWASTools' MWAS_skylineNMR on a SummarizedExperiment of NMR spectra plus an association results matrix. It maps statistical signal (sign-adjusted -log10 p-values) to spectral position (ppm) so metabolite regions of association are visually interpretable.

## When to use

Use when MWAS results from 1H NMR metabonomic data need to be visualized as a skyline plot against chemical shift (ppm), e.g. reproducing paper figures like the BMI association plot over the δ 0.80–1.60 ppm lipid region.

## When NOT to use

- Input data are mass spectrometry or other non-spectral metabolomics results — the skyline is indexed by 1H NMR chemical shift (ppm), not m/z or retention time
- The analysis goal is statistical association modeling itself (partial correlations / generalized linear models with confounder adjustment) rather than visualization of already-computed MWAS results
- A tabular or Manhattan-style summary of associations is needed rather than a spectrum-anchored skyline over a specific ppm region

## Inputs

- SummarizedExperiment object (metabo_SE) containing processed 1H NMR spectra and sample metadata including a QC reference sample (e.g. 'QC1')
- MWAS association results matrix (e.g. MWAS_BMI) with per-variable statistics (p-values / estimates) keyed to the NMR spectral variables
- Reference sample identifier string (ref_sample, e.g. 'QC1')

## Outputs

- NMR skyline plot image (e.g. MWAS_skyline_BMI.png) with sign-adjusted -log10 p-values vs chemical shift (ppm), restricted to a user-defined ppm window

## How to apply

Load the MWASTools example or user SummarizedExperiment object (metabo_SE) that holds the processed 1H NMR spectra and the QC reference sample metadata, together with the association results matrix (e.g. MWAS_BMI) produced by the package's metabolite-phenotype association models. Call MWAS_skylineNMR(metabo_SE, results_matrix, ref_sample="QC1"), which draws sign-adjusted -log10 p-values on the y-axis versus 1H NMR chemical shift in ppm on the x-axis, using the QC reference spectrum as the background trace. Restrict the displayed window to the spectral region of interest (e.g. δ 1.60–0.80 ppm for the lipid CH2/CH3 region in the BMI example) and export the figure (e.g. MWAS_skyline_BMI.png under figures/) for comparison against the published figure. The rationale is that plotting statistics in spectral order preserves metabolite-region context (peaks, shoulders, overlap) that a table of p-values loses, and the QC trace anchors each association to the underlying spectral features.

## Related tools

- **MWASTools** (Provides MWAS_skylineNMR, which renders the NMR skyline plot from the SummarizedExperiment spectra and the MWAS results matrix using a QC reference sample) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment for MWASTools (requires R >= 3.4) in which the skyline plotting call is executed)
- **rmarkdown** (Vignette engine used to render the workflow, including export of the skyline figure (MWAS_skyline_BMI.png) under figures/)

## Examples

```
MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample = "QC1")
```

## Evaluation signals

- The plot's x-axis is 1H NMR chemical shift in ppm and the displayed window matches the requested region (e.g. δ 1.60–0.80 ppm for the lipid region in the BMI example)
- The y-axis shows sign-adjusted -log10 p-values derived from the supplied MWAS results matrix (e.g. MWAS_BMI), not raw p-values
- The background skyline trace corresponds to the QC reference sample specified via ref_sample (e.g. 'QC1')
- The exported figure (e.g. MWAS_skyline_BMI.png under figures/) visually corresponds to Figure 1 of the MWASTools paper
- All spectral variables in the results matrix are represented across the ppm axis with no dropped or mismatched variables between metabo_SE and the results matrix

## Limitations

- Requires NMR data organized as a SummarizedExperiment with matching spectral variable ordering; results matrices from other pipelines may not align with the ppm axis without remapping
- Visualization only — it does not perform the association modeling or confounder adjustment; it depends on a precomputed results matrix from MWASTools' association models
- Readability degrades when the full spectral width is displayed at once; the method relies on zooming into specific ppm regions to resolve overlapping metabolite signals

## Evidence

- [full_text] Skyline plotting via MWAS_skylineNMR: [evidence span withheld — anchor status: not_found]
- [full_text] Skyline axes and ppm window: [evidence span withheld — anchor status: not_found]
- [abstract] Package visualization functionality: "Key functionalities of our package include: quality control analysis; metabolome-wide association analysis using various models (partial correlations, generalized linear models); visualization of"
- [abstract] R runtime requirement: "The MWASTools R package is implemented in R (version > =3.4)"
- [intro] Pipeline context for visualization: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis; metabolite-phenotype association models (partial correlations,"
