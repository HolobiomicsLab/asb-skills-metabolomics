---
name: xic-intensity-profile-extraction
description: Use when time-resolved, direct-injection MS1 features (e.g. from DBDI-FT-ICR-MS
  .mgf data) must be converted into per-scan XIC intensity profiles so adduct series
  such as [M+1-3O+H]+ can be correlated and mass-difference validated.
when_to_use_negative:
- Data comes from chromatographically separated LC-MS runs rather than direct-injection,
  time-resolved plasma ionization data — DBDIpy's correlation-based series logic is
  tailored to direct injection where chromatography is absent
- The feature table has not been imputed — non-detections remain np.nan and the XIC
  rows would not have uniform length, breaking pointwise correlation
- The goal is raw spectrum import or alignment itself; use align_spectra() directly
  rather than extracting XIC rows from a table that does not yet exist
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: DBDIpy
  role: Provides align_spectra() and impute_intensities() that turn aligned spectra
    into the feature DataFrame whose rows are the XIC profiles extracted here
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Imports the .mgf/.mzML/.mzXML source spectra (matchms.importing) that feed
    the aligned feature table
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Tabular structure of the feature DataFrame; XIC rows are selected with pandas
    .loc on the series feature indices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:ac84021b11d2687ea720ccf10a0e5190c0752326c0714e020bd64c887ad41a69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# xic-intensity-profile-extraction

## Summary

Extracts uniform-length extracted ion chromatogram (XIC) intensity rows — the temporal profiles across all scans — for putative in-source adduct/fragment series members from an imputed DBDIpy feature table, enabling the pointwise correlation and exact-mass-difference analysis that underpins DBDIpy's multi-ion annotation of a single analyte.

## When to use

Use when time-resolved, direct-injection MS1 features (e.g. from DBDI-FT-ICR-MS .mgf data) must be converted into per-scan XIC intensity profiles so adduct series such as [M+1-3O+H]+ can be correlated and mass-difference validated.

## When NOT to use

- Data comes from chromatographically separated LC-MS runs rather than direct-injection, time-resolved plasma ionization data — DBDIpy's correlation-based series logic is tailored to direct injection where chromatography is absent
- The feature table has not been imputed — non-detections remain np.nan and the XIC rows would not have uniform length, breaking pointwise correlation
- The goal is raw spectrum import or alignment itself; use align_spectra() directly rather than extracting XIC rows from a table that does not yet exist

## Inputs

- Raw MS1 spectra list (matchms.Spectrum objects) from an .mgf/.mzML/.mzXML file, e.g. example_dataset.mgf
- Imputed feature DataFrame (specs_imputed): rows = features, columns = scans, after align_spectra(ppm_window = 2) and impute_intensities(method = 'linear')
- identify_adducts() output dictionary of result DataFrames per adduct type (O, O2, O3, H2O) with base_index and match_index columns
- Feature metadata (e.g. example_metadata.feather) for joining annotations

## Outputs

- XIC intensity rows (temporal profiles across all scans) for the selected adduct-series features
- Combined tabulation of correlation coefficients, exact mass differences and XIC profiles per series member (e.g. oxidation_series_table.csv)

## How to apply

First build a uniform feature table by loading spectra via matchms.importing and running dbdi.align_spectra(spec, ppm_window = 2), then impute_intensities(method = 'linear') so every row is an XIC of equal length across all scans (undetected signals are np.nan before imputation). To isolate the members of an oxidation/adduct series, intersect the base_index entries of the identify_adducts() result DataFrames (e.g. O, O2, O3) with np.intersect1d and add the corresponding match_index partners — in the demo dataset this recovers the four-member series features 55, 66, 83 and 99. Extract the XIC intensity rows for those feature indices from the imputed DataFrame; these rows are what identify_adducts() correlates pointwise (all r > 0.9 for true series members). Validate series membership by checking that pairwise feature mass differences match the exact O-adduct deltas 15.99491, 31.98983 and 47.98474 Da ([M+O+H]+, [M+2O+H]+, [M+3O+H]+) within the 2 ppm mass error used in the search. Finally join annotation metadata (e.g. example_metadata.feather, annotating C15H17O2-5N) and export the correlations, mass differences and XIC profiles (e.g. to oxidation_series_table.csv).

## Related tools

- **DBDIpy** (Provides align_spectra() and impute_intensities() that turn aligned spectra into the feature DataFrame whose rows are the XIC profiles extracted here) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Imports the .mgf/.mzML/.mzXML source spectra (matchms.importing) that feed the aligned feature table) — https://github.com/matchms/matchms
- **pandas** (Tabular structure of the feature DataFrame; XIC rows are selected with pandas .loc on the series feature indices)

## Examples

```
series = np.intersect1d(np.intersect1d(res_O.base_index, res_O2.base_index), res_O3.base_index); xics = specs_imputed.loc[series]
```

## Evaluation signals

- The set of extracted feature indices equals exactly the adduct-series members recovered from intersecting result-table base_index entries (demo: features 55, 66, 83, 99)
- Every extracted XIC row has uniform length equal to the number of scans — no np.nan fields remain after impute_intensities(method = 'linear')
- Pearson correlations between the extracted XIC profiles all exceed the 0.9 threshold reported for true series members
- Pairwise feature mass differences match the exact O-adduct deltas 15.99491, 31.98983 and 47.98474 Da within the 2 ppm mass error used in the search
- Joined metadata annotations (e.g. C15H17O2-5N from example_metadata.feather) attach consistently to all tabulated series rows

## Limitations

- XIC rows contain np.nan for scans where a feature was not detected, so imputation is a prerequisite for usable profiles
- The resulting series annotation is putative and in-silico — it rests on correlation plus exact mass differences, not on confirmed structure
- Runtime optimization and advanced plotting are still under development in DBDIpy 2.0

## Evidence

- [readme] impute_intensities() will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame.: "``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame."
- [methods] pointwise correlation of XIC traces identifies adducts and fragments: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [methods] features 55 and 99 as [M+1-3O+H]+ with r > 0.9: "features 55 and 99 both putatively have [M+1-3O+H]<sup>+</sup> adduct ions with correlations of  r > 0.9 in our dataset."
- [readme] spectra become a 2D DataFrame: "By aplication of ``align_spectra()``, we transformed the list of spectra objects to a two-dimensional ``pandas.DataFrame``."
