---
name: targeted-lipidomics-transition-list-processing
description: Use when Skyline peak-area exports from PRM/SRM targeted lipidomics runs
  (e.g. Q-Exactive HF PRM or QTRAP 6500 SRM, 5 µL triplicate injections) must be QC-filtered
  and quantified for a curated lipid panel.
when_to_use_negative:
- When performing untargeted/discovery lipidomics rather than a curated targeted panel
  — LipidCreator intentionally supports knowledge-based selection of transition lists
  and does not address coeluting/isobaric lipids or peak area integration.
- When no calibration curve or technical triplicates exist, since the R² > 0.8 and
  CV < 20% QC filters cannot be computed.
- For lipid classes outside the over 60 classes supported by LipidCreator (e.g. bacterial
  acyl-PG, acyl-PE in E. coli or archaeal lipids, where standards are lacking).
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidCreator
  role: Generates the user-defined transition list and fragment library for PRM/MRM
    experiments, providing fragment definition, stable isotope labeling management,
    collision energy optimization and in silico spectral library generation.
  repo: https://github.com/lifs-tools/lipidcreator/releases
- name: Skyline
  role: Receives the transition list for method preparation, peak integration and
    peak-area data analysis of the targeted lipidomics experiment.
- name: MSStats
  role: Statistical processing of the exported peak areas for QC filtering and quantification.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:17f7d0eade242c50ac56b1f13dfa11409735fd5a8ee0dca03340cf7f8d80ad6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - MSStats
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# targeted-lipidomics-transition-list-processing

## Summary

Processes LipidCreator-generated transition lists within Skyline to quantify plasma lipid species from PRM/SRM LC-MS/MS acquisitions, applying linear-response and CV quality filters and validating quantification against NIST SRM 1950 reference material.

## When to use

Use when Skyline peak-area exports from PRM/SRM targeted lipidomics runs (e.g. Q-Exactive HF PRM or QTRAP 6500 SRM, 5 µL triplicate injections) must be QC-filtered and quantified for a curated lipid panel.

## When NOT to use

- When performing untargeted/discovery lipidomics rather than a curated targeted panel — LipidCreator intentionally supports knowledge-based selection of transition lists and does not address coeluting/isobaric lipids or peak area integration.
- When no calibration curve or technical triplicates exist, since the R² > 0.8 and CV < 20% QC filters cannot be computed.
- For lipid classes outside the over 60 classes supported by LipidCreator (e.g. bacterial acyl-PG, acyl-PE in E. coli or archaeal lipids, where standards are lacking).

## Inputs

- LipidCreator-generated transition list (Skyline-importable target list or blib spectral library)
- Skyline peak-area export from PRM/SRM acquisitions of targeted lipids
- Raw LC-MS/MS files from Q-Exactive HF (PRM) or QTRAP 6500 (SRM) runs
- Sample metadata including technical triplicate structure and calibration levels

## Outputs

- QC-filtered table of lipid species passing linear response R² > 0.8 and CV < 20% (e.g. 276 of 433 monitored species)
- Per-lipid QC metrics (R² and CV)
- Quantified lipid concentrations/abundances per sample
- Correlation result against NIST SRM 1950 reference material (target R² = 0.98)

## How to apply

Generate the transition list with LipidCreator (which computes MS/MS fragment masses for over 60 lipid classes and exports Skyline-compatible target lists or blib spectral libraries after activating the collision energy module), then import it into Skyline for method preparation and peak integration. Acquire data on a Q-Exactive HF in PRM mode or a QTRAP 6500 in SRM mode, injecting 5 µL of lipid extract resuspended in IPA/MeOH/CHCl3 (4:2:1, v/v/v) and analyzing each sample in triplicate. Export peak areas from Skyline and process them (e.g. via MSStats) by computing, per lipid species, the linear response coefficient of determination (R²) from calibration curves and the coefficient of variation (CV) across technical triplicates; retain only species meeting linear response R² > 0.8 and CV < 20%. As an external validation, join the passing species' abundances with NIST SRM 1950 standard material values and compute a direct (Pearson) correlation of concentrations; agreement should reach R² = 0.98 as observed between the American reference material and the Asian control group.

## Related tools

- **LipidCreator** (Generates the user-defined transition list and fragment library for PRM/MRM experiments, providing fragment definition, stable isotope labeling management, collision energy optimization and in silico spectral library generation.) — https://github.com/lifs-tools/lipidcreator/releases
- **Skyline** (Receives the transition list for method preparation, peak integration and peak-area data analysis of the targeted lipidomics experiment.)
- **MSStats** (Statistical processing of the exported peak areas for QC filtering and quantification.)

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- The QC-filtered species set must contain exactly 276 lipid species when applied to the plasma experiment monitoring 433 lipids in 21 healthy subjects.
- The direct Pearson correlation between quantified species and NIST SRM 1950 reference material must reproduce R² = 0.98.
- Every retained lipid species must have linear response R² > 0.8 and inter-triplicate CV < 20%.
- The exported transition list must be importable into Skyline (as a target list or blib) and yield peaks for all monitored transitions.
- Injection parameters must match the workflow: 5 µL injection volume, triplicate analysis, IPA/MeOH/CHCl3 (4:2:1) reconstitution.

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids or peak area integration.
- Retention time, elution order of lipids and isotope labeled standards need to be considered to avoid mis-annotations.
- No large-scale methods are available to cope with occurring stereoisomers due to the lack of isotope-coded standards.
- LipidCreator CE models are currently exclusive to lipid mediators; expansion to other lipid classes requires more structurally diverse standards.
- Under Linux/Mono there may be issues with repainting of certain windows and controls due to the not 100% compatible Mono implementation.

## Evidence

- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [readme] LipidCreator is a plugin for Skyline supporting targeted workflow development in lipidomics. It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline.: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
- [discussion] Samples were injected with a volume of 5 µL and analyzed in triplicates.: "Samples were injected with a volume of 5 µL and analyzed in triplicates."
- [discussion] a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
- [discussion] LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists.: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
