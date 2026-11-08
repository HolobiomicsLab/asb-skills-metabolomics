---
name: sign-adjusted-pvalue-visualization
description: Use when you have completed an MWAS on 1H NMR metabonomic data (e.g.
  BMI associations) and need to render a skyline plot of sign-adjusted p-values across
  the ppm axis, with a specific chemical-shift window such as the δ 1.60–0.80 ppm
  lipid region.
when_to_use_negative:
- When the analysis is not a metabolome-wide association on 1H NMR data and no chemical-shift
  (ppm) axis is available to plot against
- When you need to visualize associations for non-NMR data types or raw spectra rather
  than post-MWAS sign-adjusted p-values
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3172
tools:
- name: MWASTools
  role: R/Bioconductor package whose MWAS_skylineNMR function renders the sign-adjusted
    -log10 p-value skyline plot against chemical shift for NMR metabonomic MWAS results
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Language runtime in which MWASTools is implemented; the skyline visualization
    is executed as an R function call
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:7d462c55fb37a85593d7ca8fdcbc934b3e7f51476aedae9af0531ba11b3de682
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sign-adjusted-pvalue-visualization

## Summary

Visualize metabolome-wide association study (MWAS) results for 1H NMR metabonomic data as a skyline plot of sign-adjusted -log10 p-values plotted against chemical shift (ppm), using the MWASTools function MWAS_skylineNMR on a SummarizedExperiment object holding the association results and QC reference metadata.

## When to use

Use when you have completed an MWAS on 1H NMR metabonomic data (e.g. BMI associations) and need to render a skyline plot of sign-adjusted p-values across the ppm axis, with a specific chemical-shift window such as the δ 1.60–0.80 ppm lipid region.

## When NOT to use

- When the analysis is not a metabolome-wide association on 1H NMR data and no chemical-shift (ppm) axis is available to plot against
- When you need to visualize associations for non-NMR data types or raw spectra rather than post-MWAS sign-adjusted p-values

## Inputs

- SummarizedExperiment object (e.g. metabo_SE) holding MWAS results and QC reference sample metadata
- MWAS association result matrix (e.g. MWAS_BMI) with sign-adjusted p-values per metabolite/chemical shift bin
- QC reference sample identifier (e.g. ref_sample='QC1')

## Outputs

- NMR skyline plot figure (e.g. MWAS_skyline_BMI.png) showing sign-adjusted log10 p-values vs ppm, restricted to a chosen chemical-shift window

## How to apply

Load the MWASTools SummarizedExperiment object (metabo_SE) that contains the MWAS result matrix (MWAS_BMI) and QC reference sample metadata, then call MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample='QC1') to render the skyline. Plot sign-adjusted log10 p-values on the y-axis against 1H NMR chemical shift in ppm on the x-axis, restricting the displayed window to a specific region of interest — the case study uses the δ 1.60–0.80 ppm lipid region. Verify the exported figure (e.g. MWAS_skyline_BMI.png under figures/) visually corresponds to the published reference figure (Figure 1 of the paper) before treating the visualization as correct.

## Related tools

- **MWASTools** (R/Bioconductor package whose MWAS_skylineNMR function renders the sign-adjusted -log10 p-value skyline plot against chemical shift for NMR metabonomic MWAS results) — https://bioconductor.org/packages/MWASTools/
- **R** (Language runtime in which MWASTools is implemented; the skyline visualization is executed as an R function call)

## Examples

```
MWAS_skylineNMR(metabo_SE, MWAS_BMI, ref_sample="QC1")
```

## Evaluation signals

- The y-axis values correspond to sign-adjusted log10 p-values from the supplied MWAS result matrix (e.g. MWAS_BMI), not raw p-values or effect sizes
- The x-axis is 1H NMR chemical shift in ppm and the displayed window matches the intended region (case study: δ 1.60–0.80 ppm lipid region)
- The exported figure filename and location follow the vignette convention (e.g. figures/MWAS_skyline_BMI.png)
- The rendered skyline plot visually corresponds to the published reference figure (Figure 1 of the paper) when using the same metabo_SE and MWAS_BMI inputs
- The correct QC reference sample (ref_sample='QC1') was passed so that chemical-shift alignment uses the QC-based ppm axis

## Limitations

- The skyline is specific to NMR data where metabolites are indexed by chemical shift; it does not apply to mass-spectrometry or other feature types without a ppm axis
- Interpretation depends on prior MWAS quality control and confounder-adjusted association modelling, which this visualization step does not itself perform

## Evidence

- [methods] MWAS_skylineNMR call in vignette: [evidence span withheld — anchor status: not_found]
- [methods] ref_sample parameter: [evidence span withheld — anchor status: not_found]
- [methods] chemical shift axis and window: [evidence span withheld — anchor status: not_found]
- [methods] output figure name and path: [evidence span withheld — anchor status: not_found]
- [abstract] visualization as a key package functionality: "visualization of statistical outcomes"
