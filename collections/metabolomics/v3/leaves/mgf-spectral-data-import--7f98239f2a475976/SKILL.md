---
name: mgf-spectral-data-import
description: Use when raw mass spectra in .mgf (or mzML/mzXML) format must be loaded
  and converted into a feature table before correlation-based adduct/in-source fragment
  detection with DBDIpy.
when_to_use_negative:
- Your data are already a feature DataFrame (rows = features, columns = scans); DBDIpy's
  README states you can skip alignment in that case.
- The downstream goal is MS2 spectral similarity scoring (e.g. ms2deepscore) rather
  than building MS1 features for DBDI-MS adduct/in-source fragment search.
- Data come from LC-separated (chromatographic) runs rather than direct-injection,
  time-resolved plasma ionization datasets for which DBDIpy is tailored.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: matchms
  role: Provides matchms.importing.load_from_mgf (and mzML/mzXML readers) to parse
    raw spectra files into Spectrum objects before DBDIpy alignment
  repo: https://github.com/matchms/matchms
- name: DBDIpy
  role: Provides align_spectra(ppm_window=...), which converts the imported list of
    Spectrum objects into a two-dimensional pandas.DataFrame feature table
  repo: https://github.com/leopold-weidner/DBDIpy
- name: pandas
  role: Underlying tabular format of the aligned feature DataFrame (one column per
    scan, feature rows) passed to later DBDIpy functions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:a2e3b94e73a20bebb579bc65b78c0047f0e23d7dd27f5440d2ff7f6c3f66bb67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# mgf-spectral-data-import

## Summary

Imports raw MS1 spectra from open file formats (.mgf, .mzML, .mzXML) via matchms.importing and aligns them into a 2D pandas feature DataFrame with DBDIpy.align_spectra(), the entry point of the DBDIpy plasma-ionization (DBDI-MS) processing workflow.

## When to use

Use when raw mass spectra in .mgf (or mzML/mzXML) format must be loaded and converted into a feature table before correlation-based adduct/in-source fragment detection with DBDIpy.

## When NOT to use

- Your data are already a feature DataFrame (rows = features, columns = scans); DBDIpy's README states you can skip alignment in that case.
- The downstream goal is MS2 spectral similarity scoring (e.g. ms2deepscore) rather than building MS1 features for DBDI-MS adduct/in-source fragment search.
- Data come from LC-separated (chromatographic) runs rather than direct-injection, time-resolved plasma ionization datasets for which DBDIpy is tailored.

## Inputs

- MS1 spectra file in .mgf format (also .mzML or .mzXML via matchms.importing), e.g. example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089
- list of matchms Spectrum objects after load_from_mgf

## Outputs

- Two-dimensional pandas.DataFrame of aligned features: one column per mass spectrometric scan, one row per feature, first column = mean m/z, missing signals = np.nan

## How to apply

Load the .mgf file with matchms.importing.load_from_mgf into a list of matchms Spectrum objects — e.g. the demo file example_dataset.mgf from Zenodo DOI 10.5281/zenodo.7221089. Optionally run matchms preprocessing filters (noise reduction) on the spectra list before alignment. Then call DBDIpy.align_spectra(spec=spectrums, ppm_window=2): the ppm_window controls the m/z tolerance used to bin signals across scans into features. The result is a two-dimensional pandas.DataFrame with one column per MS scan and one row per feature; the first column holds the mean m/z of each feature, and scans where a signal was absent contain np.nan. If data are already formatted as such a feature DataFrame, this import/alignment step can be skipped; subsequent DBDIpy steps (impute_intensities, identify_adducts) consume the aligned table.

## Related tools

- **matchms** (Provides matchms.importing.load_from_mgf (and mzML/mzXML readers) to parse raw spectra files into Spectrum objects before DBDIpy alignment) — https://github.com/matchms/matchms
- **DBDIpy** (Provides align_spectra(ppm_window=...), which converts the imported list of Spectrum objects into a two-dimensional pandas.DataFrame feature table) — https://github.com/leopold-weidner/DBDIpy
- **pandas** (Underlying tabular format of the aligned feature DataFrame (one column per scan, feature rows) passed to later DBDIpy functions)

## Examples

```
spectrums = list(load_from_mgf(demo_mgf)); specs_aligned = dbdi.align_spectra(spec=spectrums, ppm_window=2)
```

## Evaluation signals

- load_from_mgf returns a list of matchms Spectrum objects whose count matches the expected number of scans/features in the file (the demo MGF yields the DBDI-MS feature set used downstream).
- After align_spectra, the DataFrame's first column contains mean feature m/z values and each subsequent column corresponds to one scan.
- Signals not detected in a scan appear as np.nan cells in the aligned DataFrame (before imputation).
- Setting ppm_window=2 on the demo dataset reproduces the tutorial feature table that supports the reported 710 adduct annotations (17% of features) via identify_adducts(method='pearson', threshold=0.9, mass_error=2).
- Feeding an already-aligned feature table should reproduce the same downstream impute_intensities/identify_adducts results as running the full import path.

## Limitations

- Apple M1 users may hit automatic matchms installation failures; matchms must be installed manually per its official site.
- matchms/list-of-Spectra input is required — data already formatted as a 2D feature table make this step redundant.
- Unaligned ppm_window settings will mis-bin features across scans; the tutorial value for the demo FT-ICR-MS DBDI data is ppm_window=2.

## Evidence

- [methods] align_spectra() is the interface to load data from open file formats such as .mgf, .mzML or .mzXML files via matchms.importing.: "``align_spectra()`` is the interface to load data from open file formats such as .mgf, .mzML or .mzXML files via ``matchms.importing``."
- [readme] Raw mass spectra must first be aligned to a DataFrame of features.: "Raw mass spectra containing *m/z*-intensity-pairs first will need to be aligned to a DataFrame of features."
- [readme] By application of align_spectra(), spectra objects become a 2D DataFrame with a mean m/z column and NaN for undetected signals.: "By aplication of ``align_spectra()``, we transformed the list of spectra objects to a two-dimensional ``pandas.DataFrame``."
- [readme] Tutorial loads the demo .mgf with matchms then aligns with ppm_window = 2.: "spectrums = list(load_from_mgf(demo_mgf))

##align the listed Spectra
specs_aligned = dbdi.align_spectra(spec = spectrums, ppm_window = 2)"
- [readme] If your data already is formatted accordingly, you can skip this step.: "If your data already is formatted accordingly, you can skip this step."
