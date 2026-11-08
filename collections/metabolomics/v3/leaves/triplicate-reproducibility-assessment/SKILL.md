---
name: triplicate-reproducibility-assessment
description: Use when Skyline-exported peak areas exist for a LipidCreator-generated
  transition list acquired in technical triplicates (e.g. 5 µL PRM/SRM injections)
  and you must keep only linear, reproducible lipid species before quantification
  or reference-material comparison.
when_to_use_negative:
- Peak areas come from single (non-triplicate) injections, so no CV across technical
  replicates can be computed
- The goal is untargeted/discovery lipidomics feature finding rather than QC filtering
  of a predefined LipidCreator transition list
- The task is assay design (fragment definition, collision energy optimization) rather
  than post-acquisition quantitative QC
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidCreator
  role: Generates the transition list whose measured lipid species are QC-filtered
    and quantified
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Source of the exported peak areas for the targeted lipids; LipidCreator is
    fully integrated with Skyline
- name: MSStats
  role: Statistical analysis of the quantitative targeted lipidomics results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:b66cffe21f33d69b99c4700fb304684ed364044739d3c711b92c0152f6d4d0fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - MSStats
  - Skyline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# triplicate-reproducibility-assessment

## Summary

Assesses technical reproducibility and linearity of targeted lipidomics measurements by computing per-species coefficient of variation across triplicate LC-MS/MS injections and calibration-curve R², then filtering species against QC thresholds (CV < 20%, R² > 0.8) before cross-cohort correlation against NIST SRM 1950.

## When to use

Use when Skyline-exported peak areas exist for a LipidCreator-generated transition list acquired in technical triplicates (e.g. 5 µL PRM/SRM injections) and you must keep only linear, reproducible lipid species before quantification or reference-material comparison.

## When NOT to use

- Peak areas come from single (non-triplicate) injections, so no CV across technical replicates can be computed
- The goal is untargeted/discovery lipidomics feature finding rather than QC filtering of a predefined LipidCreator transition list
- The task is assay design (fragment definition, collision energy optimization) rather than post-acquisition quantitative QC

## Inputs

- LipidCreator-generated transition list for targeted PRM/SRM assay
- Skyline-exported peak areas for 433 monitored plasma lipids from technical triplicate injections (5 µL, PRM/SRM mode on Q-Exactive HF or QTRAP 6500)
- NIST SRM 1950 standard reference material concentration values

## Outputs

- qc_summary.csv with the QC-filtered species table (276 lipid species passing R² > 0.8 and CV < 20%)
- Per-lipid QC metrics table (linear response R² and triplicate CV for each of the 433 species)
- Direct correlation result between NIST SRM 1950 and the Asian control group (expected R² = 0.98)

## How to apply

Load the LipidCreator-generated transition list and Skyline-exported peak areas for the 433 targeted plasma lipids measured in triplicate LC-MS/MS injections. For each lipid species, compute the coefficient of variation (CV) across the three technical injections and the coefficient of determination (R²) of the linear calibration response. Retain only species passing both QC thresholds — linear response R² > 0.8 AND CV < 20% — which in the plasma experiment reduces the 433 monitored species to 276. Join the passing species' abundances with NIST SRM 1950 reference values and compute the direct (Pearson) correlation of log-transformed concentrations to confirm agreement (R² = 0.98 between the American reference material and the Asian control group). Write the filtered species table, per-lipid QC metrics, and correlation result to qc_summary.csv.

## Related tools

- **LipidCreator** (Generates the transition list whose measured lipid species are QC-filtered and quantified) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Source of the exported peak areas for the targeted lipids; LipidCreator is fully integrated with Skyline)
- **MSStats** (Statistical analysis of the quantitative targeted lipidomics results)

## Examples

```
python qc_filter.py --transitions lipidcreator_transitions.csv --peaks skyline_export.csv --r2 0.8 --cv 0.20 --reference srm1950.csv --out qc_summary.csv
```

## Evaluation signals

- The QC-filtered set contains exactly 276 lipid species out of the 433 monitored in plasma
- Every retained species satisfies both thresholds: linear response R² > 0.8 and triplicate CV < 20%
- The NIST SRM 1950 vs. Asian control group direct correlation reproduces R² = 0.98
- Per-lipid QC metrics table covers all 433 monitored species with computed CV and R² before filtering
- Final output written to qc_summary.csv includes filtered species table, QC metrics, and correlation result

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids and peak area integration
- Retention time, elution order, and isotope labeled standards need to be considered to avoid mis-annotations
- Samples must be analyzed in triplicates (5 µL injection volume); fewer technical replicates invalidate the CV-based filter

## Evidence

- [discussion] Samples were injected with a volume of 5 µL and analyzed in triplicates.: "Samples were injected with a volume of 5 µL and analyzed in triplicates."
- [discussion] Direct correlation R² = 0.98 between reference material and Asian control group: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
- [readme] LipidCreator integrates with Skyline for targeted PRM/MRM assay development: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
- [discussion] NIST SRM 1950 used as standard reference material for plasma analysis: "by performing an analysis of human plasma lipidome samples and NIST SRM 1950 standard material"
