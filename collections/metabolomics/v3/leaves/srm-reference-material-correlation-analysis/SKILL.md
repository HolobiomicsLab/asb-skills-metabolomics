---
name: srm-reference-material-correlation-analysis
description: Use when targeted LC-MS/MS peak areas (e.g., Skyline-exported, from PRM/SRM
  runs) must be benchmarked against the NIST SRM 1950 plasma reference material to
  demonstrate quantitative agreement across cohorts or labs.
when_to_use_negative:
- The input is untargeted lipidomics data without Skyline peak areas or a LipidCreator
  transition list — the QC filtering depends on targeted PRM/SRM triplicate measurements
- No reference material (e.g., NIST SRM 1950) values are available to join against,
  so a correlation benchmark cannot be computed
- The goal is FDR estimation or isobaric/coeluting lipid resolution — LipidCreator
  explicitly does not provide a dedicated scoring algorithm or address coeluting/isobaric
  lipids and peak area integration
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidCreator
  role: Generates the targeted transition list used to quantify the monitored plasma
    lipids before QC filtering and correlation
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Performs peak-area extraction for the PRM/SRM transitions whose areas feed
    the R² and CV QC filtering
- name: MSStats
  role: Statistical analysis of the targeted lipidomics quantification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:f6f6a0fdf5907caca27d18367802df6528ef9157e84054e6d9b3fbe8b067ab75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - MSStats
  - Skyline
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# srm-reference-material-correlation-analysis

## Summary

Quantitative cross-lab agreement assessment that correlates lipid abundances measured in a study cohort (here, plasma from 21 healthy Asian subjects quantified via a LipidCreator/Skyline PRM/SRM assay) against the NIST SRM 1950 standard reference material values, verifying inter-population/inter-lab agreement by Pearson R².

## When to use

Use when targeted LC-MS/MS peak areas (e.g., Skyline-exported, from PRM/SRM runs) must be benchmarked against the NIST SRM 1950 plasma reference material to demonstrate quantitative agreement across cohorts or labs.

## When NOT to use

- The input is untargeted lipidomics data without Skyline peak areas or a LipidCreator transition list — the QC filtering depends on targeted PRM/SRM triplicate measurements
- No reference material (e.g., NIST SRM 1950) values are available to join against, so a correlation benchmark cannot be computed
- The goal is FDR estimation or isobaric/coeluting lipid resolution — LipidCreator explicitly does not provide a dedicated scoring algorithm or address coeluting/isobaric lipids and peak area integration

## Inputs

- LipidCreator-generated transition list for targeted plasma lipids (433 species)
- Skyline-exported peak areas from triplicate LC-MS/MS injections (5 µL, PRM/SRM mode) of 21 healthy Asian subject plasma samples
- NIST SRM 1950 standard reference material concentration values for human plasma lipids

## Outputs

- Filtered lipid species table containing exactly 276 species passing R² > 0.8 and CV < 20% QC filters
- Per-lipid QC metrics (linear-response R² and triplicate CV) written to qc_summary.csv
- Pearson correlation result between NIST SRM 1950 and the Asian control group (expected R² = 0.98)

## How to apply

Load the LipidCreator-generated transition list and Skyline-exported peak areas for the targeted lipids (433 plasma lipids measured as 5 µL injections in technical triplicate, PRM/SRM mode). For each lipid species compute the linear-response coefficient of determination (R²) from the calibration curve and the coefficient of variation (CV) across triplicate injections, retaining only species passing both QC thresholds (R² > 0.8 and CV < 20%); this yields the 276-species filtered set. Join the passing species' measured abundances with the NIST SRM 1950 standard reference material values, log-transform concentrations, and compute the direct (Pearson) correlation between the American reference material and the Asian control group. A correct analysis reproduces R² = 0.98, confirming that the Asian control plasma agrees well with the American reference material.

## Related tools

- **LipidCreator** (Generates the targeted transition list used to quantify the monitored plasma lipids before QC filtering and correlation) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Performs peak-area extraction for the PRM/SRM transitions whose areas feed the R² and CV QC filtering)
- **MSStats** (Statistical analysis of the targeted lipidomics quantification)

## Examples

```
python qc_correlate.py --transitions lipidcreator_transitions.csv --areas skyline_peak_areas.csv --reference NIST_SRM_1950.csv --r2-threshold 0.8 --cv-threshold 20 --out qc_summary.csv
```

## Evaluation signals

- The QC-filtered species set contains exactly 276 lipid species after applying R² > 0.8 and CV < 20% to the 433 monitored lipids
- The Pearson correlation between NIST SRM 1950 and the Asian control group log-transformed concentrations reaches R² = 0.98
- qc_summary.csv contains per-lipid QC metrics (linear-response R² and triplicate CV) for all 433 monitored species
- All retained species have CV computed across technical triplicate injections of the same 5 µL injection volume runs
- Every retained species has a finite calibration-curve R² from the linear response model

## Limitations

- Retention time, elution order, and isotope labeled standards must be considered to avoid mis-annotations, which could corrupt the correlation input
- LipidCreator does not address coeluting/isobaric lipids or peak area integration, so mis-integrated peaks propagate into the correlation
- Direct correlation reflects relative agreement only; it does not validate absolute concentration accuracy for individual lipid classes

## Evidence

- [discussion] Direct correlation analysis reveals R2 = 0.98 between the American reference material and the Asian control group: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
- [methods] Samples injected in triplicate with 5 µL volume: "Samples were injected with a volume of 5 µL and analyzed in triplicates."
- [methods] PRM or SRM acquisition on Q-Exactive HF or QTRAP 6500: "The LC was coupled to a Q-Exactive HF (QEx HF) mass spectrometer (Thermo Scientific, Bremen, Germany) or a QTRAP 6500 (Applied Biosystems, Darmstadt, Germany) to use PRM or SRM acquisition mode"
- [discussion] NIST SRM 1950 analyzed alongside human plasma lipidome samples: "by performing an analysis of human plasma lipidome samples and NIST SRM 1950 standard material"
- [discussion] LipidCreator does not address coeluting/isobaric lipids or peak area integration: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists. Thus, it intentionally does not address"
