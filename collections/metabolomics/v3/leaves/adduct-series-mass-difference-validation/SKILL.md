---
name: adduct-series-mass-difference-validation
description: 'Use when identify_adducts() output from a direct-injection plasma ionization
  (DBDI) MS dataset must be checked: candidate features should share a base analyte
  and differ by exact O-adduct deltas.'
when_to_use_negative:
- Data acquired with chromatographic separation (LC-MS), where adduct grouping should
  rely on co-elution rather than the time-resolved XIC correlation/mass-difference
  logic designed for direct-injection plasma ionization
- Input is a feature table without XIC intensities across scans, so pointwise correlation
  of temporal profiles cannot be computed
- Candidate m/z differences cannot match any user-defined or default adduct rule (e.g.
  [M-H2O+H]+, [M+1O+H]+, [M+2O+H]+ or custom O3 delta 47.984744 Da) within the ppm_window
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DBDIpy
  role: Provides identify_adducts() (pointwise correlation of XIC traces plus exact
    mass-difference analysis) and align_spectra()/impute_intensities() that produce
    the feature DataFrame being validated
  repo: https://github.com/leopold-weidner/DBDIpy
- name: pandas
  role: Holds the aligned features as a 2D DataFrame whose rows (features) and columns
    (scans) are intersected and tabulated during validation
- name: matchms
  role: Imports the .mgf/.mzML/.mzXML source spectra via matchms.importing before
    alignment and adduct-search refinement
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:f5b25ee69b1c039bacdae90960f58f970a17b5c9e63df28b74a2d4c2f3aed662
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# adduct-series-mass-difference-validation

## Summary

Validates a putative in-source oxidation/adduct series from DBDI-MS data by checking that candidate features match the exact mass deltas of O-adduct species ([M+O+H]+, [M+2O+H]+, [M+3O+H]+) within the search ppm window, while also exhibiting correlated temporal profiles.

## When to use

Use when identify_adducts() output from a direct-injection plasma ionization (DBDI) MS dataset must be checked: candidate features should share a base analyte and differ by exact O-adduct deltas.

## When NOT to use

- Data acquired with chromatographic separation (LC-MS), where adduct grouping should rely on co-elution rather than the time-resolved XIC correlation/mass-difference logic designed for direct-injection plasma ionization
- Input is a feature table without XIC intensities across scans, so pointwise correlation of temporal profiles cannot be computed
- Candidate m/z differences cannot match any user-defined or default adduct rule (e.g. [M-H2O+H]+, [M+1O+H]+, [M+2O+H]+ or custom O3 delta 47.984744 Da) within the ppm_window

## Inputs

- identify_adducts() output dictionary (one DataFrame per adduct rule: O, O2, H2O, O3) with base_index and match_index columns
- Imputed feature DataFrame (specs_imputed) of uniform-length XICs from align_spectra(ppm_window=2) and impute_intensities(method='linear')
- Metadata table (example_metadata.feather) with analyte annotations
- Feature mean m/z values (first column of the aligned feature DataFrame)

## Outputs

- Combined table of correlation coefficients, exact mass differences and XIC intensity profiles for validated series members (e.g. oxidation_series_table.csv)
- Confirmed set of base features and their oxidized partners (e.g. features 55, 66, 83, 99 for [C15H17O2N+H]+)

## How to apply

Load the per-adduct DataFrames returned by DBDIpy's identify_adducts() (one each for O, O2, H2O, O3 rules) and intersect their base_index entries with np.intersect1d to recover base features whose XIC traces correlate with multiple oxidized partners. For each candidate pair, verify that the observed mean m/z difference equals the exact O-adduct delta — 15.99491 Da for [M+O+H]+, 31.98983 Da for [M+2O+H]+, 47.98474 Da for [M+3O+H]+ (e.g. custom O3 rule deltamz = 47.984744) — within the ppm_window used in alignment (2 ppm in the tutorial). Also confirm the Pearson correlation of the XIC temporal profiles exceeds the 0.9 threshold, since DBDIpy's identification relies on pointwise correlation of time-resolved traces plus exact mass differences. Finally, join metadata (example_metadata.feather) to confirm the annotation of the base analyte, and export the combined correlation, mass-difference and XIC-intensity table (e.g. oxidation_series_table.csv).

## Related tools

- **DBDIpy** (Provides identify_adducts() (pointwise correlation of XIC traces plus exact mass-difference analysis) and align_spectra()/impute_intensities() that produce the feature DataFrame being validated) — https://github.com/leopold-weidner/DBDIpy
- **pandas** (Holds the aligned features as a 2D DataFrame whose rows (features) and columns (scans) are intersected and tabulated during validation)
- **matchms** (Imports the .mgf/.mzML/.mzXML source spectra via matchms.importing before alignment and adduct-search refinement) — https://github.com/matchms/matchms

## Examples

```
series = np.intersect1d(np.intersect1d(results['O'].base_index, results['O2'].base_index), results['O3'].base_index); xics = specs_imputed.loc[series]
```

## Evaluation signals

- The validated feature set is exactly the intersected series members (e.g. features 55, 66, 83, 99) recovered from O/O2/O3 base_index intersections
- Every pairwise Pearson correlation coefficient of XIC traces exceeds the 0.9 threshold
- Observed m/z differences match the exact O-adduct deltas 15.99491, 31.98983 and 47.98474 Da within the 2 ppm mass error of the search
- Metadata join confirms all members annotate to the same base analyte (e.g. C15H17O2-5N)
- XIC profiles are uniform in length across all scans after linear imputation, so correlation is computed over complete traces

## Limitations

- By default identify_adducts() only searches [M-H2O+H]+, [M+1O+H]+ and [M+2O+H]+; broader series (e.g. [M+3O+H]+) require user-defined custom adduct rules such as deltamz = 47.984744
- Identifications are putative only — correlation plus mass difference does not guarantee the annotated oxidation series, especially in direct-injection data lacking chromatographic separation
- Apple M1 users may need to install the matchms dependency manually before the workflow can run
- The validation inherits the alignment tolerance: features must have been aligned with a tight ppm_window (2 ppm in the tutorial) for the exact-delta check to be meaningful

## Evidence

- [methods] By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>.: "By default, ``identify_adducts()`` searches for [M-H<sub>2</sub>O+H]<sup>+</sup>, [M+1O+H]<sup>+</sup> and [M+2O+H]<sup>+</sup>."
- [methods] This tells us that features 55 and 99 both putatively have [M+1-3O+H]<sup>+</sup> adduct ions with correlations of  r > 0.9 in our dataset.: "This tells us that features 55 and 99 both putatively have [M+1-3O+H]<sup>+</sup> adduct ions with correlations of  r > 0.9"
- [intro] (exact) mass differences are used to refine the nature of potential candidates.: "(exact) mass differences are used to refine the nature of potential candidates."
- [readme] calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment.: "calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment."
- [methods] ``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame.: "``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC)"
