---
name: xic-missing-value-imputation
description: Use when an aligned DBDI-MS feature table (one column per scan, one row
  per feature) contains np.nan fields from scans where a signal was not detected,
  and downstream identify_adducts() analysis requires uniform-length XICs.
when_to_use_negative:
- Input is already a raw list of matchms.Spectra or an unaligned spectra file — align_spectra()
  must be applied first; impute_intensities() expects the 2D aligned feature DataFrame.
- The aligned table contains no missing values (specs_aligned.isnull().values.any()
  is False) — imputation is unnecessary and baseline padding could distort complete
  traces.
- The downstream analysis does not require uniform-length XICs, e.g. pure spectrum-level
  matchms preprocessing pipelines with optional noise-reduction filters that bypass
  feature-table construction.
tools:
- name: DBDIpy
  role: Provides impute_intensities(), the two-stage interpolation-plus-baseline imputation
    function that produces uniform-length XICs
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Imports the .mgf/.mzML/.mzXML spectra that align_spectra() converts into the
    feature table passed to impute_intensities()
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Holds the 2D feature DataFrame whose missing values impute_intensities() fills;
    used for isnull()/describe()/info() checks
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:c63415059aaffc2ba419a6f96997aacd914c9edcc1c751992f6dcb99fb3c9612
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# xic-missing-value-imputation

## Summary

DBDIpy's impute_intensities() performs two-stage missing-value imputation on aligned DBDI-MS feature tables: it interpolates missing intensities within each feature's signal region (method='linear') and pads with a noisy baseline, yielding uniform-length extracted ion chromatograms (XICs) ready for pointwise-correlation adduct/fragment detection.

## When to use

Use when an aligned DBDI-MS feature table (one column per scan, one row per feature) contains np.nan fields from scans where a signal was not detected, and downstream identify_adducts() analysis requires uniform-length XICs.

## When NOT to use

- Input is already a raw list of matchms.Spectra or an unaligned spectra file — align_spectra() must be applied first; impute_intensities() expects the 2D aligned feature DataFrame.
- The aligned table contains no missing values (specs_aligned.isnull().values.any() is False) — imputation is unnecessary and baseline padding could distort complete traces.
- The downstream analysis does not require uniform-length XICs, e.g. pure spectrum-level matchms preprocessing pipelines with optional noise-reduction filters that bypass feature-table construction.

## Inputs

- Aligned DBDI-MS feature table (pandas.DataFrame) from DBDIpy.align_spectra(spec, ppm_window=2) with one column per scan, features as rows, a 'mean' m/z column, and np.nan for undetected signals
- Preprocessed list of matchms.Spectra objects loaded from .mgf/.mzML/.mzXML (e.g. example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089)
- Imputation method parameter (e.g. 'linear')

## Outputs

- Imputed pandas.DataFrame of uniform-length XICs with no residual NaNs
- Persisted artifact such as specs_imputed.feather for downstream identify_adducts() / propose_adducts() runs

## How to apply

First load .mgf spectra via matchms.importing.load_from_mgf and align them with dbdi.align_spectra(spec=..., ppm_window=2), which produces a pandas.DataFrame with one column per scan, a 'mean' m/z column, and np.nan wherever a signal was absent in a scan; drop the 'mean' column into a separate feature_mz Series. Confirm gaps exist with specs_aligned.isnull().values.any() and inspect via pandas describe()/info(). Then call dbdi.impute_intensities(df=specs_aligned, method='linear'): the two-stage routine interpolates missing intensities within each feature's detected signal region, then adds a noisy baseline outside it so every XIC matches the length of the longest XIC in the dataset. Rationale: pointwise correlation in identify_adducts() needs intensity traces of equal length, so linear interpolation within signal regions preserves temporal profiles while baseline padding avoids fabricating correlations. Validate no residual NaNs, then persist the result (e.g. specs_imputed.feather) before running identify_adducts() with method='pearson', threshold=0.95 and mass_error=2.

## Related tools

- **DBDIpy** (Provides impute_intensities(), the two-stage interpolation-plus-baseline imputation function that produces uniform-length XICs) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Imports the .mgf/.mzML/.mzXML spectra that align_spectra() converts into the feature table passed to impute_intensities()) — https://github.com/matchms/matchms
- **pandas** (Holds the 2D feature DataFrame whose missing values impute_intensities() fills; used for isnull()/describe()/info() checks)

## Examples

```
import DBDIpy as dbdi; specs_imputed = dbdi.impute_intensities(df=specs_aligned, method='linear')
```

## Evaluation signals

- specs_imputed.isnull().values.any() returns False — no residual NaNs after imputation.
- On the demo dataset (Zenodo DOI 10.5281/zenodo.7221089), the aligned input table contains the expected 500 features; the imputed output preserves this row count.
- Every imputed XIC has the same length, equal to the longest XIC in the dataset (uniform-length check across all columns/rows).
- Interpolated values stay within each feature's detected signal region; padded regions carry noisy-baseline intensity rather than zero or inflated signal, so downstream pointwise correlation (r > 0.95, mass_error=2 ppm in identify_adducts) behaves as reported (710 adducts ≈ 17% of 4196 demo features).
- Result can be round-tripped losslessly via feather (specs_imputed.feather) with the same shape and dtypes as the in-memory DataFrame.

## Limitations

- Imputation interpolates only within a feature's detected signal region; outside it, values are noisy baseline, so temporal profiles outside signal regions are synthetic and could mislead correlation-based searches if signal-region detection fails.
- align_spectra() places np.nan in fields where a signal was not detected in a scan; if the ppm_window (e.g. 2) is mis-set, real detections may be lost to NaN and increase imputation burden.
- Apple M1 chip users might encounter issues with automatic installation of matchms, a prerequisite dependency of this step; manual installation per the matchms site is required.

## Evidence

- [methods] impute_intensities(): two-stage imputation of missing values: "``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame."
- [readme] np.nan for undetected signals: "If a signal was not detected in a scan, the according field will be set to an instance of ``np.nan``."
- [readme] alignment prerequisite: "Raw mass spectra containing *m/z*-intensity-pairs first will need to be aligned to a DataFrame of features. We build features by using the ``align_spectra()`` function."
- [intro] downstream correlation search: "calculation of pointwise correlation identifies features with matching temporal intensity profiles through the experiment."
