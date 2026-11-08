---
name: direct-infusion-ms-feature-alignment
description: Use when input is raw .mgf, .mzML or .mzXML spectra from direct injection
  / plasma ionization (DBDI) MS with no LC separation, and a row-per-feature, column-per-scan
  DataFrame is needed before imputation or adduct detection.
when_to_use_negative:
- Input is already an aligned feature table — align_spectra() can be skipped per the
  tutorial ('If your data already is formatted accordingly, you can skip this step').
- Data comes from chromatography-coupled LC-MS where features are defined by retention
  time alignment rather than direct-infusion scan-wise m/z alignment.
- A ppm_window mismatched to instrument accuracy is required — mass alignment tolerance
  must reflect the mass spectrometer's error (2 ppm was appropriate for FT-ICR, not
  for low-resolution instruments).
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DBDIpy
  role: Provides align_spectra(), which transforms a list of matchms spectra objects
    into a feature-aligned pandas.DataFrame (features as rows, scans as columns, mean
    m/z in the first column).
  repo: https://github.com/leopold-weidner/DBDIpy
- name: matchms
  role: Interface for importing raw spectra from .mgf/.mzML/.mzXML via matchms.importing.load_from_mgf
    and for optional preprocessing (e.g. noise reduction) before alignment.
  repo: https://github.com/matchms/matchms
- name: pandas
  role: Underlying 2D tabular data structure produced by align_spectra(); feature
    table rows/columns are pandas.DataFrame fields consumed by downstream DBDIpy functions.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad088
    title: 'DBDIpy: a Python library for processing of untargeted datasets from real-time
      plasma ionization mass spectrometry'
  source_content_hash: sha256:35f08f17a13106821834366051a0a67e244ae18be934b00ecf37f9c02ef7b96c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leopold-weidner/DBDIpy
  tools:
  - DBDIpy
  - Python
  - matchms
  - pandas
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# direct-infusion-ms-feature-alignment

## Summary

Aligns raw mass spectra from direct-infusion (no chromatographic separation) MS experiments — e.g. DBDI-FT-ICR-MS — into a 2D feature table of m/z rows versus scan columns using DBDIpy's align_spectra() over matchms-imported spectra. This feature table is the prerequisite for imputation and adduct/in-source fragment detection.

## When to use

Use when input is raw .mgf, .mzML or .mzXML spectra from direct injection / plasma ionization (DBDI) MS with no LC separation, and a row-per-feature, column-per-scan DataFrame is needed before imputation or adduct detection.

## When NOT to use

- Input is already an aligned feature table — align_spectra() can be skipped per the tutorial ('If your data already is formatted accordingly, you can skip this step').
- Data comes from chromatography-coupled LC-MS where features are defined by retention time alignment rather than direct-infusion scan-wise m/z alignment.
- A ppm_window mismatched to instrument accuracy is required — mass alignment tolerance must reflect the mass spectrometer's error (2 ppm was appropriate for FT-ICR, not for low-resolution instruments).

## Inputs

- list of matchms.Spectra objects loaded from example_dataset.mgf (or .mzML/.mzXML) via matchms.importing
- ppm_window parameter controlling feature alignment tolerance (e.g. 2 ppm for FT-ICR-MS data)

## Outputs

- 2D pandas.DataFrame with one row per aligned feature, one column per MS scan, first column = mean feature m/z, missing signals = np.nan
- aligned spectra object (specs_aligned) ready for impute_intensities() and identify_adducts()

## How to apply

Import the raw spectra into a list of matchms.Spectra objects with matchms.importing.load_from_mgf (or load_from_mzml/load_from_mzxml for other open formats); optionally run matchms preprocessing such as noise reduction at this stage. Then call DBDIpy's align_spectra(spec=spectrums, ppm_window=2), which aligns m/z-intensity pairs across scans into a two-dimensional pandas.DataFrame where features are rows, scans are columns, and the first column holds each feature's mean m/z. Signals absent from a scan are set to np.nan, which downstream impute_intensities() resolves into uniform-length extracted ion chromatograms (XIC). The ppm_window controls the mass tolerance used to merge signals into one feature and must be set to match instrument mass accuracy (the demo used 2 ppm on FT-ICR data); after alignment the table can be parsed directly to identify_adducts() for combined pointwise-correlation and mass-difference analysis.

## Related tools

- **DBDIpy** (Provides align_spectra(), which transforms a list of matchms spectra objects into a feature-aligned pandas.DataFrame (features as rows, scans as columns, mean m/z in the first column).) — https://github.com/leopold-weidner/DBDIpy
- **matchms** (Interface for importing raw spectra from .mgf/.mzML/.mzXML via matchms.importing.load_from_mgf and for optional preprocessing (e.g. noise reduction) before alignment.) — https://github.com/matchms/matchms
- **pandas** (Underlying 2D tabular data structure produced by align_spectra(); feature table rows/columns are pandas.DataFrame fields consumed by downstream DBDIpy functions.)

## Examples

```
from matchms.importing import load_from_mgf; import DBDIpy as dbdi; spectrums = list(load_from_mgf('example_dataset.mgf')); specs_aligned = dbdi.align_spectra(spec=spectrums, ppm_window=2)
```

## Evaluation signals

- Output is a pandas.DataFrame with one row per feature and one column per mass spectrometric scan, with the first column showing the mean m/z of each feature.
- Undetected signals appear as np.nan instances, not zeros, ready for impute_intensities() imputation.
- Feature count is consistent with the input spectra: the demo dataset yields 500 randomly selected features from the wheat-bread DBDI-FT-ICR-MS experiment.
- Alignment tolerance matches instrument accuracy (ppm_window=2 was used in the demo/tutorial); mean m/z values should be stable within that window across scans.

## Limitations

- Apple M1 chip users might encounter issues with automatic installation of matchms; manual installation of the dependency is required to resolve the issue.
- DBDIpy requires Python 3.7, 3.8, 3.9 or 3.10 (Anaconda recommended); other Python versions are unsupported.
- Data from direct injection lacks chromatographic separation, which complicates identification of fragments and adducts — alignment alone is only putative and must be followed by correlation/mass-difference (and optionally MS2 similarity) refinement.

## Evidence

- [methods] align_spectra() is the interface to load data from open file formats such as .mgf, .mzML or .mzXML files via matchms.importing.: "``align_spectra()`` is the interface to load data from open file formats such as .mgf, .mzML or .mzXML files via ``matchms.importing``."
- [readme] By application of align_spectra(), we transformed the list of spectra objects to a two-dimensional pandas.DataFrame.: "By aplication of ``align_spectra()``, we transformed the list of spectra objects to a two-dimensional ``pandas.DataFrame``. Now you have a column for each mass spectrometric scan and features are"
- [methods] Missing values are np.nan and imputation yields uniform-length XICs.: "``impute_intensities()`` will assure that after imputation we will have a set of uniform length extracted ion chromatograms (XIC) in our DataFrame."
- [readme] Demo alignment invocation with ppm_window=2 from the tutorial.: "specs_aligned = dbdi.align_spectra(spec = spectrums, ppm_window = 2)"
