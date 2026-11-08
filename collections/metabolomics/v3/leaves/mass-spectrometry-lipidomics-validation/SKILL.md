---
name: mass-spectrometry-lipidomics-validation
description: Use when a targeted lipidomics transition list (PRM/SRM) has been generated
  in silico — e.g., LipidCreator-computed fragment masses for defined fatty acyl combinations
  — and you must confirm the computed ion signatures yield linear instrument response
  before deploying the assay.
when_to_use_negative:
- The goal is untargeted/discovery lipid screening rather than quantitative validation
  of a defined subset of lipid targets.
- The assay needs false discovery rate scoring or resolution of coeluting/isobaric
  lipids and peak-area integration — LipidCreator intentionally does not address these.
- The lipidome is bacterial or archaeal (e.g., E. coli acyl-PG/acyl-PE) or requires
  stereoisomer separation, where standards are lacking and coverage is incomplete.
tools:
- name: LipidCreator
  role: Computes the MS/MS fragment masses and SRM/MRM transition list whose ion signatures
    are being validated; fully supports targeted lipidomics assay development
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts the LipidCreator transition list, runs the targeted assay, and supplies
    peak-area results used as the calibration response variable
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: flipR
  role: R framework used to fit fragment-specific response/dissociation profiles and
    linear calibration models per lipid class
- name: minpack.lm
  role: Levenberg–Marquardt nonlinear least-squares solver used within flipR for the
    regression fits
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Bounded grid-search optimization to initialize/stabilize the nonlinear fits
  repo: https://rdrr.io/cran/nls.multstart/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:08a78886450b408078c23378a6b8a0b74857e1d807faa0e0dc0a57b7b45feb7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - Skyline
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-spectrometry-lipidomics-validation

## Summary

Validate that computationally derived lipid ion signatures (MS/MS transition lists from LipidCreator) produce the expected linear MS response in a targeted lipidomics assay, by fitting per-lipid-class calibration curves of LC/ESI SRM peak-area response versus amount and checking correlation coefficients against a reported benchmark (average r ≈ 0.99 for the 18:0–20:4 PUFA-PL series in human platelet matrix).

## When to use

Use when a targeted lipidomics transition list (PRM/SRM) has been generated in silico — e.g., LipidCreator-computed fragment masses for defined fatty acyl combinations — and you must confirm the computed ion signatures yield linear instrument response before deploying the assay.

## When NOT to use

- The goal is untargeted/discovery lipid screening rather than quantitative validation of a defined subset of lipid targets.
- The assay needs false discovery rate scoring or resolution of coeluting/isobaric lipids and peak-area integration — LipidCreator intentionally does not address these.
- The lipidome is bacterial or archaeal (e.g., E. coli acyl-PG/acyl-PE) or requires stereoisomer separation, where standards are lacking and coverage is incomplete.

## Inputs

- LipidCreator-generated transition list (MRM/PRM targets) for the fatty acyl combination across PA, PC, PE, PG, PI, PS
- Skyline peak-area response data from LC/ESI SRM runs in human platelet lipid matrix
- Lipid extraction samples (MeOH/MTBE extract, resuspended in IPA/MeOH/CHCl3 4:2:1 v/v/v, 5 µL injections, triplicates on QTRAP 6500)

## Outputs

- Per-lipid-class linear calibration curves (peak-area response vs. amount)
- Per-class Pearson correlation coefficients recorded in platelet_response_correlations.csv
- Average correlation coefficient across the six lipid classes (benchmark: 0.99)

## How to apply

Load the LipidCreator-generated transition list (e.g., the 18:0–20:4 fatty acyl combination across PA, PC, PE, PG, PI, and PS) together with Skyline peak-area results from SRM runs in the biological matrix (human platelet lipid extract). Import the Skyline transition results into R and organize fragment-specific response values by headgroup. For each lipid class, fit linear regression of peak-area response versus amount using the flipR nonlinear-regression framework with the minpack.lm Levenberg–Marquardt solver and nls.multstart bounded grid-search optimization. Compute the Pearson correlation coefficient for the linear response region of each class and average across classes (PA, PC, PE, PG, PI, PS). Validation passes when the average coefficient reproduces the reported benchmark (0.99), demonstrating LipidCreator computes correct ion signatures suitable for targeted assays.

## Related tools

- **LipidCreator** (Computes the MS/MS fragment masses and SRM/MRM transition list whose ion signatures are being validated; fully supports targeted lipidomics assay development) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts the LipidCreator transition list, runs the targeted assay, and supplies peak-area results used as the calibration response variable) — https://skyline.ms/project/home/software/Skyline/begin.view
- **flipR** (R framework used to fit fragment-specific response/dissociation profiles and linear calibration models per lipid class)
- **minpack.lm** (Levenberg–Marquardt nonlinear least-squares solver used within flipR for the regression fits) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Bounded grid-search optimization to initialize/stabilize the nonlinear fits) — https://rdrr.io/cran/nls.multstart/

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- Average Pearson correlation coefficient across PA, PC, PE, PG, PI, PS equals the reported 0.99 benchmark
- Each per-class fit is linear in the linear response region (per-class r values recorded in platelet_response_correlations.csv)
- Observed fragment masses match the LipidCreator-computed transition list; fragment masses cross-validated against reference resources such as the Metlin database
- Skyline peak-area integration completes for all transitions with 5 µL triplicate injections yielding monotonic response vs. amount
- Cross-material sanity check: direct correlation analysis of quantified lipids between reference and control material gives good agreement (R2 = 0.98 for NIST SRM 1950 vs. Asian control group)

## Limitations

- LipidCreator's collision-energy models are currently exclusive to lipid mediators; extending CE prediction to other lipid classes requires more structurally diverse standards and CE-dependent fragmentation measurements.
- LipidCreator provides no dedicated scoring algorithm for false discovery rates and intentionally does not address coeluting/isobaric lipids or peak-area integration.
- Retention time, elution order, and isotope-labeled standards must be considered to avoid mis-annotations; no large-scale stereoisomer-resolved methods exist due to lack of isotope-coded standards.
- Eukaryotic model systems are covered with some exceptions; rare bacterial/archaeal lipid classes (e.g., E. coli acyl-PG, acyl-PE) lack standards.

## Evidence

- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [discussion] We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm): "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm)"
- [discussion] Nonlinear Regression Model: flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles: "Nonlinear Regression Model: flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] The LC was coupled to a Q-Exactive HF (QEx HF) mass spectrometer (Thermo Scientific, Bremen, Germany) or a QTRAP 6500 (Applied Biosystems, Darmstadt, Germany) to use PRM or SRM acquisition mode: "The LC was coupled to a Q-Exactive HF (QEx HF) mass spectrometer (Thermo Scientific, Bremen, Germany) or a QTRAP 6500 (Applied Biosystems, Darmstadt, Germany) to use PRM or SRM acquisition mode"
- [readme] LipidCreator is a plugin for Skyline supporting targeted workflow development in lipidomics.: [evidence span withheld — anchor status: not_found]
- [discussion] Samples were injected with a volume of 5 µL and analyzed in triplicates.: "Samples were injected with a volume of 5 µL and analyzed in triplicates."
