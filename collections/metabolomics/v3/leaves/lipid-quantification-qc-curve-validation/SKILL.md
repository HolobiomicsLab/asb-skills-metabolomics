---
name: lipid-quantification-qc-curve-validation
description: Use when a LipidCreator-generated transition list has been measured as
  PRM/SRM triplicate LC-MS/MS injections (e.g., 5 µL, Q-Exactive HF or QTRAP 6500)
  and you must quantify targeted plasma lipids, filter species by curve linearity
  and precision, and compare to NIST SRM 1950.
when_to_use_negative:
- The analysis is untargeted/screening or DIA-based without a LipidCreator-designed
  transition list and calibration curves — LipidCreator is for targeted lipidomics
  assay development and intentionally does not address peak area integration or coeluting/isobaric
  lipids
- Input is already a QC-filtered feature table with no replicate injections, so per-species
  CV < 20% and calibration-curve R² > 0.8 cannot be computed
- The analytes are not lipids (e.g., lipid mediators outside the covered classes or
  bacterial/archaeal lipids lacking standards), since the CE models cover only a subset
  of lipid classes and the majority of eukaryotic lipid classes have some exceptions
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipidCreator
  role: Generates the user-defined target lists and fragment libraries (transition
    lists) for the PRM/MRM targeted quantification experiment that this QC workflow
    validates
  repo: github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Used for DIA method preparation and targeted data analysis; exports the peak
    areas per lipid species that feed the R² and CV computations
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: MSStats
  role: Downstream statistical analysis of the Skyline-exported quantification results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:4f8b8c85df371a29c0167a401cad42cc9d930c6348509c04833df3fc26a8143e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - MSStats
  - Skyline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lipid-quantification-qc-curve-validation

## Summary

QC-validates a LipidCreator/Skyline targeted lipidomics quantification by retaining only lipid species with linear response R² > 0.8 and triplicate CV < 20%, then benchmarking quantification against NIST SRM 1950 reference material via log-transformed Pearson correlation.

## When to use

Use when a LipidCreator-generated transition list has been measured as PRM/SRM triplicate LC-MS/MS injections (e.g., 5 µL, Q-Exactive HF or QTRAP 6500) and you must quantify targeted plasma lipids, filter species by curve linearity and precision, and compare to NIST SRM 1950.

## When NOT to use

- The analysis is untargeted/screening or DIA-based without a LipidCreator-designed transition list and calibration curves — LipidCreator is for targeted lipidomics assay development and intentionally does not address peak area integration or coeluting/isobaric lipids
- Input is already a QC-filtered feature table with no replicate injections, so per-species CV < 20% and calibration-curve R² > 0.8 cannot be computed
- The analytes are not lipids (e.g., lipid mediators outside the covered classes or bacterial/archaeal lipids lacking standards), since the CE models cover only a subset of lipid classes and the majority of eukaryotic lipid classes have some exceptions

## Inputs

- LipidCreator-generated transition list for targeted lipids (e.g., 433 plasma lipid species)
- Skyline-exported peak areas from triplicate LC-MS/MS injections (5 µL, PRM or SRM mode)
- NIST SRM 1950 standard reference material concentration values

## Outputs

- QC-filtered lipid species table (species passing R² > 0.8 and CV < 20%; 276 of 433 in the reference run)
- Per-lipid QC metrics (linear response R², CV across triplicates) in qc_summary.csv
- Pearson correlation result between measured abundances and NIST SRM 1950 values (expected R² = 0.98)

## How to apply

Load the LipidCreator-generated transition list and the Skyline-exported peak areas for the targeted lipids (here 433 species measured in triplicate 5 µL injections in PRM/SRM mode). For each lipid species compute the coefficient of determination (R²) of the calibration curve and the coefficient of variation (CV) across the technical triplicates. Apply the dual QC threshold — retain only species with linear response R² > 0.8 AND CV < 20% — which for the 433 monitored plasma lipids in 21 healthy Asian subjects yields exactly 276 passing species. Join the passing species' abundances with NIST SRM 1950 standard reference material values and compute a direct Pearson correlation on log-transformed concentrations; agreement should reach R² = 0.98 between the American reference material and the Asian control group. Write the filtered species table, per-lipid QC metrics, and the correlation result to qc_summary.csv.

## Related tools

- **LipidCreator** (Generates the user-defined target lists and fragment libraries (transition lists) for the PRM/MRM targeted quantification experiment that this QC workflow validates) — github.com/lifs-tools/lipidcreator
- **Skyline** (Used for DIA method preparation and targeted data analysis; exports the peak areas per lipid species that feed the R² and CV computations) — https://skyline.ms/project/home/software/Skyline/begin.view
- **MSStats** (Downstream statistical analysis of the Skyline-exported quantification results)

## Examples

```
mono LipidCreator.exe
```

## Evaluation signals

- The QC-filtered set contains exactly 276 lipid species when starting from the 433 monitored plasma lipids
- Every retained species satisfies both thresholds: calibration-curve linear response R² > 0.8 and triplicate CV < 20%
- The log-transformed concentration correlation between NIST SRM 1950 and the Asian control group reproduces R² = 0.98
- qc_summary.csv exists and contains the filtered species table, per-lipid QC metrics, and the correlation result
- Peak-area input is triplicate data (each sample injected with a volume of 5 µL and analyzed in triplicates), so CV is computable for all species

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids or peak area integration
- Retention time, elution order of lipids, and isotope-labeled standards must be considered to avoid mis-annotations
- Collision-energy models are currently exclusive to lipid mediators; expansion to other lipid classes requires more structurally diverse standards and CE-dependent measurements
- Due to the lack of isotope-coded standards, there are currently no large-scale methods to cope with occurring stereoisomers

## Evidence

- [discussion] Direct correlation reveals R² = 0.98 between reference material and Asian controls: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
- [discussion] Triplicate 5 µL injections underpin CV computation: "Samples were injected with a volume of 5 µL and analyzed in triplicates."
- [discussion] PRM/SRM acquisition modes used for the targeted assay: "The LC was coupled to a Q-Exactive HF (QEx HF) mass spectrometer (Thermo Scientific, Bremen, Germany) or a QTRAP 6500 (Applied Biosystems, Darmstadt, Germany) to use PRM or SRM acquisition mode"
- [readme] LipidCreator creates the transition lists used by this QC workflow: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
- [discussion] Skyline handles the targeted data analysis that produces the peak areas: "DIA method preparation and data analysis were performed with Skyline (See Supplementary Methods)."
