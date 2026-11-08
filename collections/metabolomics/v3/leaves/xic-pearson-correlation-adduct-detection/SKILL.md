---
name: xic-pearson-correlation-adduct-detection
description: Use when a direct-injection or plasma-ionization (DBDI) MS dataset has
  been aligned and imputed into uniform-length XIC feature tables and you need to
  putatively identify adducts/in-source fragments of the same analyte without chromatographic
  separation.
when_to_use_negative:
- Data still consists of raw unaligned mass spectra — alignment (align_spectra) and
  imputation (impute_intensities) must be done first to obtain uniform-length XICs.
- Data comes from chromatography-separated LC-MS rather than time-resolved direct-injection/plasma
  ionization data; the method relies on pointwise correlation of temporally co-varying
  XIC traces within a single experiment.
- You need confirmed (not putative) adduct annotations — the output is only putative
  identification of multiple ion species without chromatographic separation.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DBDIpy
  role: Provides identify_adducts() implementing the combined pointwise correlation
    and mass-difference search, plus align_spectra() and impute_intensities() for
    the required upstream preprocessing
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Loads .mgf/.mzML/.mzXML files into matchms.Spectra objects via matchms.importing
    (load_from_mgf) before alignment; DBDIpy is integrated into the matchms ecosystem
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Holds the 2D feature table (one column per scan, rows as features) consumed
    and returned by DBDIpy functions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:8b9629d9ecf2ace1f95aa92675c5185cf9043a83c27d4aab1a8144e851c9b8bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# XIC Pearson Correlation Adduct Detection

## Summary

Detects in-source adducts and fragments (e.g. [M-H2O+H]+, [M+O+H]+, [M+2O+H]+) in direct-injection plasma ionization MS datasets by combining pointwise Pearson correlation of extracted ion chromatograms with exact mass-difference filtering, as implemented in DBDIpy.identify_adducts().

## When to use

Use when a direct-injection or plasma-ionization (DBDI) MS dataset has been aligned and imputed into uniform-length XIC feature tables and you need to putatively identify adducts/in-source fragments of the same analyte without chromatographic separation.

## When NOT to use

- Data still consists of raw unaligned mass spectra — alignment (align_spectra) and imputation (impute_intensities) must be done first to obtain uniform-length XICs.
- Data comes from chromatography-separated LC-MS rather than time-resolved direct-injection/plasma ionization data; the method relies on pointwise correlation of temporally co-varying XIC traces within a single experiment.
- You need confirmed (not putative) adduct annotations — the output is only putative identification of multiple ion species without chromatographic separation.

## Inputs

- Imputed XIC feature DataFrame (pandas.DataFrame, one column per scan, uniform-length extracted ion chromatograms) from DBDIpy.impute_intensities()
- Feature mean m/z values (e.g. the 'mean' m/z column of the aligned DataFrame)
- Optional custom adduct rules for the open search
- Input spectra previously loaded from .mgf/.mzML/.mzXML via matchms.importing (e.g. example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089)

## Outputs

- Dictionary of per-adduct-type result DataFrames with columns base_mz, base_index, match_mz, match_index, mzdiff, corr
- Exported CSV of identified adducts (e.g. identified_adducts.csv)
- Putative adduct/in-source fragment annotations for a subset of features (e.g. 710 adducts ≈ 17% of features in the demo dataset)

## How to apply

After aligning spectra with DBDIpy.align_spectra(spec=..., ppm_window=2) and imputing missing values with DBDIpy.impute_intensities(df=..., method='linear'), run DBDIpy.identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.95, mass_error=2). The two-step search first computes pointwise Pearson correlation of XIC traces to find features with matching temporal intensity profiles (r > 0.95), then refines candidates by exact mass differences (e.g. 18.01057 for +H2O, 15.99491 for +O, 31.98983 for +2O) within a 2 ppm mass error window. Default adduct rules search for [M-H2O+H]+, [M+1O+H]+ and [M+2O+H]+; custom adduct rules can be passed for the open search. Export the returned per-adduct-type result DataFrames (columns base_mz, base_index, match_mz, match_index, mzdiff, corr) for downstream visualization or CSV reporting.

## Related tools

- **DBDIpy** (Provides identify_adducts() implementing the combined pointwise correlation and mass-difference search, plus align_spectra() and impute_intensities() for the required upstream preprocessing) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Loads .mgf/.mzML/.mzXML files into matchms.Spectra objects via matchms.importing (load_from_mgf) before alignment; DBDIpy is integrated into the matchms ecosystem) — https://github.com/matchms/matchms
- **pandas** (Holds the 2D feature table (one column per scan, rows as features) consumed and returned by DBDIpy functions)

## Examples

```
adducts = dbdi.identify_adducts(df=specs_imputed, masses=feature_mz, method='pearson', threshold=0.95, mass_error=2)
```

## Evaluation signals

- On the wheat bread DBDI-FT-ICR-MS demo dataset (4196 features), the pipeline should identify 710 potential adducts, corresponding to ~17% of all features.
- Returned mzdiff values should match exact adduct deltas: 18.01057 for +H2O, 15.99491 for +O, 31.98983 for +2O.
- All reported correlation coefficients should exceed the chosen threshold (r > 0.95 with method='pearson').
- Mass matches should fall within the configured 2 ppm mass error window.
- Result DataFrames should contain the expected columns: base_mz, base_index, match_mz, match_index, mzdiff, corr; the exported identified_adducts.csv must exist.

## Limitations

- Identifications are only putative — the in-silico approach identifies candidate ion species but does not confirm them.
- Interpretation of direct-injection data is inherently ambiguous because missing chromatographic separation complicates identification of fragments and adducts.
- The search refines candidates by correlation and mass difference; MS2 spectral similarity scoring refinement is a newer direction (V2) and the three-step procedure was still in preparation.
- Apple M1 chip users may encounter installation issues with matchms, requiring manual installation before the pipeline can run.

## Evidence

- [readme] DBDIpy's core functionality relies on putative identification of in-source fragments and adducts via correlation and mass differences: [evidence span withheld — anchor status: not_found]
- [methods] Default adduct rules searched by identify_adducts: [evidence span withheld — anchor status: not_found]
- [methods] Two-step search rationale: correlation then mass difference: "we compute pointwise correlation of XIC traces to identify in-source adducts or in-source fragments generated during the plasma ionization process."
- [methods] Demo result: features 55 and 99 with adduct correlations r > 0.9: [evidence span withheld — anchor status: not_found]
