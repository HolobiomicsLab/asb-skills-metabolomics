---
name: ion-mobility-lipidomics-quality-control
description: Use when LC-IM-MS lipidomics measurements (e.g. trapped IM-MS or DTIM-MS)
  are spiked with fully U13C-labeled yeast lipids and CCS bias against a DTCCSN2 reference
  must be quantified or corrected per lipid class-adduct combination.
when_to_use_negative:
- Samples were not spiked with U13C-labeled lipids, so no 'heavy' labeled entries
  exist to match against the DTCCSN2 library.
- Data were not acquired by IM-MS or CCS values are unavailable in the import table
  — bias calculation and correction depend entirely on measured CCS values.
- Lipid class-adduct combination of interest is not among the 10 correctable classes
  (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG), so CCS correction (not bias calculation)
  cannot be performed.
edam_operation: http://edamontology.org/operation_3481
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MobiLipid
  role: R Markdown tool that performs CCS bias calculation and lipid class-adduct-specific
    CCS correction against the U13C-labeled lipid DTCCSN2 library
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (version 4.3.2) in which MobiLipid's automated CCS monitoring
    and correction R Markdown is executed
  repo: https://cran.r-project.org/
- name: RStudio
  role: IDE (2023.09.1) used to run MobiLipid, choose input files via pop-up dialogs,
    and render the .html/.pdf outputs
- name: Skyline
  role: Processes LC-IM-MS raw data — LC peak integration using the ECN model, ion
    mobility library creation, and IM filtering — before CCS QC
- name: PNNL PreProcessor
  role: Demultiplexes and smooths multiplexed LC-DTIM-MS data prior to CCS calibration
    and bias assessment
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:eacdc8c4d3e60bc284fcd83d1209141405299d0176e3d930ae6b65ae4a22611d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - PNNL PreProcessor
  - R
  - R Markdown
  - RStudio
  - Skyline
  edam_operation: http://edamontology.org/operation_3481
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ion-mobility-lipidomics-quality-control

## Summary

Internal CCS quality control and bias correction for IM-MS lipidomics using a DTCCSN2 library of U13C-labeled yeast lipids as internal standards, implemented as the MobiLipid R Markdown workflow.

## When to use

Use when LC-IM-MS lipidomics measurements (e.g. trapped IM-MS or DTIM-MS) are spiked with fully U13C-labeled yeast lipids and CCS bias against a DTCCSN2 reference must be quantified or corrected per lipid class-adduct combination.

## When NOT to use

- Samples were not spiked with U13C-labeled lipids, so no 'heavy' labeled entries exist to match against the DTCCSN2 library.
- Data were not acquired by IM-MS or CCS values are unavailable in the import table — bias calculation and correction depend entirely on measured CCS values.
- Lipid class-adduct combination of interest is not among the 10 correctable classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG), so CCS correction (not bias calculation) cannot be performed.

## Inputs

- Measured lipid CCS table as .csv with headers File, LipidClass, LipidSpecies, Adduct, Label, CCS
- U13C_DT_CCS_library.csv — DTCCSN2 library of 377 values for U13C-labeled yeast lipids
- Samples spiked with fully U13C-labeled yeast extract internal standards

## Outputs

- CCS_bias_no_correction.csv — per-lipid CCS bias (%) before correction
- Corrected_CCS_values.csv and Corrected_CCS_values_mean.csv with post-correction bias per correction function
- Correction_functions.csv listing all generated linear CCS correction functions
- .html and .pdf report with bias tables, resampling summaries, and violin plots
- .RData file storing all results as a list

## How to apply

Spike samples with U13C-labeled yeast extract internal standards, process the LC-IM-MS data (Skyline 23.1.0.268, PNNL PreProcessor 4.12 demultiplexing for Agilent 6560 DTIM-MS, IM-MS Browser 10.0 calibration), and export a .csv with headers File, LipidClass, LipidSpecies, Adduct, Label ('light'/'heavy'), CCS. In R/RStudio run MobiLipid_CCS-bias-calculation.Rmd with U13C_DT_CCS_library.csv, matching entries on identical lipid class/species/adduct nomenclature and restricting to valid class-adduct combinations (e.g. PC with [M+H], [M+Na], [M+HCOO]; TG with [M+NH4], [M+Na]). CCS bias is the relative deviation of measured CCS from the library DTCCSN2 value; correction uses linear regression (y = m*x + b) with measured U13C-labeled CCS as x and library values as y, requiring a minimum of 3 lipids per class-adduct combination and computing up to 100 resampled functions from 3–6 lipids. Corrected CCS values are validated by re-computing bias of corrected values against library values of the U13C-labeled lipids.

## Related tools

- **MobiLipid** (R Markdown tool that performs CCS bias calculation and lipid class-adduct-specific CCS correction against the U13C-labeled lipid DTCCSN2 library) — github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (version 4.3.2) in which MobiLipid's automated CCS monitoring and correction R Markdown is executed) — https://cran.r-project.org/
- **RStudio** (IDE (2023.09.1) used to run MobiLipid, choose input files via pop-up dialogs, and render the .html/.pdf outputs)
- **Skyline** (Processes LC-IM-MS raw data — LC peak integration using the ECN model, ion mobility library creation, and IM filtering — before CCS QC)
- **PNNL PreProcessor** (Demultiplexes and smooths multiplexed LC-DTIM-MS data prior to CCS calibration and bias assessment)

## Examples

```
Rmd <- tcltk::tk_choose.files(caption = "Select R markdown (.Rmd file) which should be used for data processing:"); rmarkdown::render(Rmd)
```

## Evaluation signals

- Mean absolute CCSN2 bias between TIM-MS and the DTCCSN2 reference should be ~0.78% (positive) and ~0.33% (negative) for U13C-labeled lipids, and ~0.73% / ~0.46% for unlabeled lipids.
- After CCS correction, mean absolute biases of the 10 lipid classes should be reduced to approximately 0%.
- Every corrected CCS value must derive from a linear function built from ≥3 U13C-labeled lipids in the same lipid class-adduct combination.
- Import .csv must contain exactly the headers File, LipidClass, LipidSpecies, Adduct, Label, CCS with adducts limited to [M+H], [M+NH4], [M+Na], [M-H], [M+HCOO].
- A systematic DTCCSN2 shift of ~0.25% between U13C-labeled and unlabeled lipid pairs (Supporting Information Figure S5) is expected when comparing light/heavy matches.

## Limitations

- CCS correction is restricted to 10 lipid classes (Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG) because linear regression requires ≥3 lipids per class-adduct combination; other classes support bias calculation only.
- Correlation of CCS with mass is not linear for some lipid classes (e.g. PC, [M+H], [M+Na]; TG, [M+NH4]), which can weaken linear correction functions.
- Not all adducts are valid for each lipid class — only the documented combinations (e.g. SPH with [M+H] only) are supported.
- U13C-labeled and unlabeled lipids show a systematic DTCCSN2 shift of approximately 0.25% attributable to reduced mass at moderate IM resolution.
- Tables in the .pdf output show only the first 10 rows; full tables must be read from the .html file.

## Evidence

- [readme] MobiLipid allows for the correction of CCS values by computing correction function for each lipid class-adduct combinations: "MobiLipid allows for the correction of CCS values by computing correction function for each lipid class-adduct combinations"
- [readme] Linear correction requires minimum of 3 lipids per class-adduct combination: "linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination"
- [readme] CCS correction restricted to 10 lipid classes: "restricts the CCS correction to the following lipid classes: Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, and TG"
- [readme] MobiLipid computes up to 100 distinct correction functions from 3–6 lipids: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression"
- [readme] Regression inputs: measured U13C-labeled CCS as x, library DTCCSN2 as y: "Input values for linear regression using the equation *y* = *m* \* *x* + *b* are measured CCS values of U<sup>13</sup>C labeled lipids as x value and <sup>DT</sup>CCS<sub>N2</sub> library values as y"
- [readme] Required import headers for the measured data table: "The .csv file has to have the following headers: “File”, “LipidClass”, “LipidSpecies”, “Adduct”, “Label”, “CCS”"
- [abstract] Reported mean absolute CCSN2 biases between TIM-MS and DTCCSN2 reference: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [abstract] CCS correction reduces biases to approximately 0%: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [intro] Library construction filter criteria including CCS repeatability: "(4) a CCS relative standard deviation <1%"
- [supplementary] Systematic DTCCSN2 shift between labeled and unlabeled lipids: [evidence span withheld — anchor status: not_found]
- [intro] Platform and tool versions used for CCS quality control: "further data filtering, evaluation, and visualization was done in the R studio programming environment (R version 4.3.2 and RStudio 2023.09.1)"
