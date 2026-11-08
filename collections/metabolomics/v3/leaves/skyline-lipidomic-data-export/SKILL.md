---
name: skyline-lipidomic-data-export
description: Use when Skyline holds a LipidCreator transition list for targeted PRM/MRM
  lipid measurements and peak areas must be exported (e.g., 433 plasma lipids in triplicate)
  for QC filtering or reference-material correlation.
when_to_use_negative:
- The assay is untargeted DDA or discovery lipidomics rather than a targeted LipidCreator
  transition list in Skyline
- The task requires false discovery rate scoring or automated handling of coeluting/isobaric
  lipids and peak area integration, which LipidCreator intentionally does not address
- The lipid class lacks structurally diverse standards (e.g., acyl-PG/acyl-PE in E.
  coli or archaeal lipids), so collision energy model predictions are unavailable
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: LipidCreator
  role: Generates the user-defined target lists and fragment libraries for PRM/MRM
    experiments that Skyline exports; also exports .blib spectral libraries
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts the targeted lipid assay, processes the PRM/SRM raw files in triplicate,
    and exports peak areas for downstream QC and quantification
- name: MSStats
  role: Statistical reporting on the exported peak areas to derive per-lipid R² and
    CV QC metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:e0ae7f61d7a53c1420f9e78353f6f199b6628eea7a8ff5e8b22600b506cc7bb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - MSStats
  - Skyline
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# skyline-lipidomic-data-export

## Summary

Exports peak areas and transition data for a LipidCreator-generated targeted lipidomics assay from Skyline (PRM on Thermo Q-Exactive HF or SRM on QTRAP 6500) so downstream QC metrics (R², CV) and quantitative comparisons against NIST SRM 1950 can be computed.

## When to use

Use when Skyline holds a LipidCreator transition list for targeted PRM/MRM lipid measurements and peak areas must be exported (e.g., 433 plasma lipids in triplicate) for QC filtering or reference-material correlation.

## When NOT to use

- The assay is untargeted DDA or discovery lipidomics rather than a targeted LipidCreator transition list in Skyline
- The task requires false discovery rate scoring or automated handling of coeluting/isobaric lipids and peak area integration, which LipidCreator intentionally does not address
- The lipid class lacks structurally diverse standards (e.g., acyl-PG/acyl-PE in E. coli or archaeal lipids), so collision energy model predictions are unavailable

## Inputs

- LipidCreator-generated transition list (targets and fragments for PRM/MRM)
- Skyline document with imported LC-MS/MS raw files (Q-Exactive HF PRM or QTRAP 6500 SRM, 5 µL injections, technical triplicates)
- NIST SRM 1950 standard reference material values for plasma lipids
- Lipid extracts resuspended in IPA/MeOH/CHCl3 (4:2:1, v/v/v) or MeOH (fatty acids)

## Outputs

- Skyline-exported peak-area table for 433 monitored lipid species across triplicate injections
- QC-filtered species table (276 species passing R² > 0.8 and CV < 20%) and per-lipid QC metrics written to qc_summary.csv
- Correlation result (R² = 0.98) between NIST SRM 1950 and the Asian control group
- Optional .blib spectral library export

## How to apply

Import the LipidCreator-generated transition list into Skyline and process the vendor raw files acquired in triplicate with 5 µL injections (PRM on Q-Exactive HF or SRM on QTRAP 6500). Export per-species peak areas for all monitored lipids (433 in the plasma experiment) from the Skyline document. Pass the exported table to MSStats/Skyline statistical reporting to compute per-lipid linear-response R² from calibration curves and CV across technical triplicates. Retain only species meeting both QC thresholds (R² > 0.8 and CV < 20%) — the plasma study expects exactly 276 passing species. Join passing species' abundances with NIST SRM 1950 reference values and verify the log-transformed correlation reaches R² = 0.98 between the American reference material and the Asian control group before writing qc_summary.csv. Spectral libraries can additionally be exported as a .blib file from the lipid review dialog after activating the collision energy module and adding transitions.

## Related tools

- **LipidCreator** (Generates the user-defined target lists and fragment libraries for PRM/MRM experiments that Skyline exports; also exports .blib spectral libraries) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts the targeted lipid assay, processes the PRM/SRM raw files in triplicate, and exports peak areas for downstream QC and quantification)
- **MSStats** (Statistical reporting on the exported peak areas to derive per-lipid R² and CV QC metrics)

## Evaluation signals

- The QC-filtered species table contains exactly 276 lipid species out of 433 monitored
- The NIST SRM 1950 vs. Asian control group correlation reproduces R² = 0.98
- Exported peak areas cover 21 healthy subject plasma samples measured in technical triplicate with 5 µL injection volume
- Every retained species satisfies both thresholds: linear response R² > 0.8 and CV < 20%
- A .blib export (after activating the collision energy module) succeeds, confirming sqlite interop is functional on Linux builds

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm for false discovery rates and intentionally does not address coeluting/isobaric lipids or peak area integration
- Retention time, elution order, and isotope-labeled standards must be considered to avoid mis-annotations; no large-scale stereoisomer methods exist due to lack of isotope-coded standards
- On Linux, .blib export requires sqlite3 and may need a custom-built libSQLite.Interop.so; Mono has imperfect window repainting and LipidCreator has only been tested with Thermo QExactive HF and Agilent QTOF instruments

## Evidence

- [readme] LipidCreator is a plugin for Skyline supporting targeted workflow development in lipidomics.: "LipidCreator is a plugin for [Skyline](https://skyline.ms/project/home/software/Skyline/begin.view) supporting targeted workflow development in lipidomics."
- [readme] It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline.: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
- [methods] Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [discussion] Samples were injected with a volume of 5 µL and analyzed in triplicates.: "Samples were injected with a volume of 5 µL and analyzed in triplicates."
- [discussion] a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
