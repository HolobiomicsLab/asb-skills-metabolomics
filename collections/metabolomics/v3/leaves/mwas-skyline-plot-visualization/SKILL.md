---
name: mwas-skyline-plot-visualization
description: Use when MWAS results (association/p-value matrix for a phenotype such
  as BMI) are available for 1H NMR metabonomic data and you need to visualize which
  spectral regions (ppm windows) carry significant associations.
when_to_use_negative:
- When the data are not spectral (ppm-axis) — e.g. results from metabolite-level feature
  tables without chemical shifts, where a standard Manhattan/volcano plot is more
  appropriate
- When the goal is statistical testing or model fitting itself rather than visualization
  of already-computed MWAS results
- For mass spectrometry metabolomics peak plots, since MWAS_skylineNMR is specific
  to 1H NMR spectral representation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MWASTools
  role: Provides MWAS_skylineNMR, which renders MWAS results as an NMR skyline plot
    from the metabo_SE SummarizedExperiment and the MWAS results matrix
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which MWASTools (version >= 3.4) and the skyline plotting
    function are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:8372826349395a4751c946ac3e005cdecde72c7d18de62aae02c9f64994c89d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mwas-skyline-plot-visualization

## Summary

Render metabolome-wide association study (MWAS) results over 1H NMR spectra as a 'skyline' plot using MWASTools' MWAS_skylineNMR, plotting sign-adjusted -log10 p-values against chemical shift in ppm so statistically significant metabolite-phenotype associations can be visually localized to spectral regions.

## When to use

Use when MWAS results (association/p-value matrix for a phenotype such as BMI) are available for 1H NMR metabonomic data and you need to visualize which spectral regions (ppm windows) carry significant associations.

## When NOT to use

- When the data are not spectral (ppm-axis) — e.g. results from metabolite-level feature tables without chemical shifts, where a standard Manhattan/volcano plot is more appropriate
- When the goal is statistical testing or model fitting itself rather than visualization of already-computed MWAS results
- For mass spectrometry metabolomics peak plots, since MWAS_skylineNMR is specific to 1H NMR spectral representation

## Inputs

- SummarizedExperiment object with 1H NMR metabonomic data (e.g. MWASTools example metabo_SE)
- MWAS results matrix for a phenotype (e.g. MWAS_BMI) containing statistics such as estimates and p-values
- QC reference sample identifier (e.g. ref_sample = "QC1") for spectral alignment

## Outputs

- Skyline plot of sign-adjusted -log10 p-values vs chemical shift (ppm), e.g. MWAS_skyline_BMI.png
- Figure restricted to a chosen ppm window (e.g. delta 1.60-0.80 ppm lipid region)

## How to apply

Load the SummarizedExperiment object (e.g. MWASTools' example metabo_SE) holding the NMR spectral data and QC reference sample metadata, together with the MWAS results matrix (e.g. MWAS_BMI produced by the package's partial correlation or generalized linear model association functions). Call MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample="QC1") so the statistical signal is plotted in spectral space rather than by variable name. Plot sign-adjusted log10 p-values on the y-axis against the 1H NMR chemical shift in ppm on the x-axis, and restrict the displayed window to the region of interest (e.g. the delta 1.60-0.80 ppm lipid region) to make peaks interpretable. Export the figure (e.g. as MWAS_skyline_BMI.png under figures/) and compare it against the corresponding published figure to confirm the visualization is correct.

## Related tools

- **MWASTools** (Provides MWAS_skylineNMR, which renders MWAS results as an NMR skyline plot from the metabo_SE SummarizedExperiment and the MWAS results matrix) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which MWASTools (version >= 3.4) and the skyline plotting function are executed)

## Examples

```
MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample = "QC1")
```

## Evaluation signals

- The plot's x-axis is 1H NMR chemical shift in ppm and the y-axis contains sign-adjusted log10 p-values (positive values for positive associations, negative for inverse)
- The rendered skyline visually corresponds to Figure 1 of the MWASTools paper when run on the metabo_SE example data with MWAS_BMI
- The ppm window matches the requested region (e.g. delta 1.60-0.80 ppm for the lipid region) rather than the full spectrum
- The figure file is exported to the expected path (figures/MWAS_skyline_BMI.png) without errors from the ref_sample alignment step

## Limitations

- Requires a QC reference sample (ref_sample) present in the SummarizedExperiment metadata for spectral alignment; results are unreliable if QC spectra are absent
- Skyline visualization depends on prior computation of a valid MWAS results matrix (e.g. via the package's partial correlation or GLM models); it does not itself compute associations
- Interpretation of peaks is limited to spectral regions; metabolite assignment requires complementary functionality such as STOCSY

## Evidence

- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [intro] visualization of statistical outcomes is a key functionality: "visualization of statistical outcomes; and metabolite assignment using Statistical Total Correlation Spectroscopy (STOCSY)"
- [methods] MWAS_skylineNMR call on metabo_SE: [evidence span withheld — anchor status: not_found]
