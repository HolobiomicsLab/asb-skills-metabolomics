---
name: moving-window-interpolation-baseline-correction
description: Use when raw multi-platform CE-MS/HILIC-MS data converted to mzXML or
  mzML has been ROI-searched and false-positive-filtered in AriumMS, and retained
  ROIs still carry baseline drift that must be removed before feature detection and
  integration.
when_to_use_negative:
- Input is already a processed feature/intensity table (e.g. post-XCMS Online, MetaboAnalyst,
  MetaboAnalystR, or MZmine feature detection) — baseline correction belongs before
  feature detection in the AriumMS pipeline
- Data need MS/MS spectral networking or feature linking across methods — AriumMS
  explicitly does not implement this yet (listed as future development)
- Single-platform targeted quantification where baseline correction at the ROI stage
  is unnecessary and direct peak integration suffices
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: MATLAB app whose preprocessing stage applies the moving-window interpolation
    baseline correction to ROI-searched, false-positive-filtered data
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Prerequisite conversion of vendor MS data to mzXML/mzML, the format AriumMS
    ROI search and preprocessing consume
  repo: http://proteowizard.sourceforge.net/download.html
- name: ZMat
  role: Compression toolbox dependency required on the MATLAB path when running the
    AriumMS app that performs the correction
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:b66cf20053ccf2f9474e9f79cd9444770f7919e94a1e13fc4386cadfabf8b56f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ZMat
  - msconvert (ProteoWizard)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# moving-window-interpolation-baseline-correction

## Summary

Baseline correction step from the AriumMS untargeted metabolomics toolbox: after ROI search and false-positive filtering, a moving-window interpolation method removes low-frequency baseline signal from retained ROIs in CE-MS and HILIC-MS (LC-MS) data so that feature detection and integration operate on background-free intensities.

## When to use

Use when raw multi-platform CE-MS/HILIC-MS data converted to mzXML or mzML has been ROI-searched and false-positive-filtered in AriumMS, and retained ROIs still carry baseline drift that must be removed before feature detection and integration.

## When NOT to use

- Input is already a processed feature/intensity table (e.g. post-XCMS Online, MetaboAnalyst, MetaboAnalystR, or MZmine feature detection) — baseline correction belongs before feature detection in the AriumMS pipeline
- Data need MS/MS spectral networking or feature linking across methods — AriumMS explicitly does not implement this yet (listed as future development)
- Single-platform targeted quantification where baseline correction at the ROI stage is unnecessary and direct peak integration suffices

## Inputs

- mzXML or mzML files produced by msconvert from CE-MS or HILIC-MS Orbitrap acquisitions
- ROI search results from AriumMS (m/z, scan-based region, raw intensity) that survived the isotope/adduct/contaminant false-positive filter
- User-defined AriumMS processing parameters (intensity threshold, minimum consecutive-scan ROI size, moving-window settings) entered via the GUI

## Outputs

- Baseline-corrected ROI table (m/z, scan-based region, corrected intensity) passed to AriumMS feature detection and integration
- Preprocessed per-method intensity matrices ready for augmentation (mid-level data fusion) and chemometric evaluation

## How to apply

Convert vendor MS files (three CE-Orbitrap MS with nanoCEasy interface, two HILIC-Orbitrap MS in the reference study) to .mzXML or .mzML with msconvert (ProteoWizard), since AriumMS runs in MATLAB R2024a or newer with Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes. Run ROI search with a user-defined intensity threshold and minimum ROI size in consecutive MS1 scans, then remove isotope, adduct, and common-contaminant candidates. Apply moving-window interpolation baseline correction to the retained ROI intensities within the AriumMS preprocessing stage (ROIpeaks/MSroiaug functions from the Tauler/Gorrochategui/Jaumot protocol); the rationale is that narrow CE-MS peaks sit on sloping backgrounds, and uncorrected baseline inflates apparent peak areas and degrades downstream mid-level data fusion and chemometrics. The corrected ROI table (m/z, scan-based region, baseline-corrected intensity) is then fed to feature detection and integration. Judge correctness by verifying that ground-truth/standard-mixture ROIs retain expected intensities while empty or noise-only scan windows flatten toward zero.

## Related tools

- **AriumMS** (MATLAB app whose preprocessing stage applies the moving-window interpolation baseline correction to ROI-searched, false-positive-filtered data) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Prerequisite conversion of vendor MS data to mzXML/mzML, the format AriumMS ROI search and preprocessing consume) — http://proteowizard.sourceforge.net/download.html
- **ZMat** (Compression toolbox dependency required on the MATLAB path when running the AriumMS app that performs the correction) — http://github.com/fangq/zmat

## Examples

```
msconvert sample.raw -o . --mzXML  # then load the mzXML in the AriumMS MATLAB app (R2024a+) and run ROI search -> preprocessing (moving-window interpolation baseline correction)
```

## Evaluation signals

- Synthetic mzXML ground-truth peaks with known ROI signals survive correction with expected intensities, while background-noise-only windows flatten to near-zero baseline
- No negative or drift-inflated intensities remain: corrected peak areas should match known concentrations in the metabolic standard mixtures
- Corrected ROI tables per method (three CE-MS, two HILIC-MS) feed augmentation without platform-specific offset artifacts
- Downstream chemometrics (e.g. yeast treated vs. untreated with growth inhibitor) separates phenotypes only when baseline correction was applied, per the paper's benchmark

## Limitations

- Raw paper data (CE-MS/LC-MS of standards and yeast extracts) are not publicly deposited, so correction parameters must be tuned on your own data or synthetic inputs
- No version number or changelog is available for AriumMS, and no explicit code/license availability statement appears in the section quoted
- Correction quality depends on the ROI search parameters (intensity threshold, minimum consecutive MS1 scans) set upstream; poor ROI candidates propagate through preprocessing

## Evidence

- [discussion] AriumMS pipeline stages: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [readme] Underlying ROI functions: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot https://doi.org/10.1038/protex.2015.102"
- [readme] Parameter-driven ROI search and preprocessing: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation."
- [readme] File format conversion prerequisite: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project http://proteowizard.sourceforge.net/download.html"
