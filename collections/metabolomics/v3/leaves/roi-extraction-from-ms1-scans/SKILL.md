---
name: roi-extraction-from-ms1-scans
description: Use when raw CE-MS or LC-MS (mzXML/mzML) MS1 data must be reduced to
  candidate ROIs (m/z, scan-based region, intensity) prior to feature detection, especially
  for multi-platform metabolomics augmentation.
when_to_use_negative:
- Input is already a preprocessed feature table rather than raw mzXML/mzML MS1 data
  — ROI search would be redundant
- Data are MS/MS-level spectra or targeted MS2 acquisitions; AriumMS ROI search operates
  on MS1 scans and MS/MS spectral networking is not implemented
- Single-platform analysis where existing tools like XCMS Online, MetaboAnalyst, MetaboAnalystR,
  or MZmine already suffice
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: AriumMS
  role: Primary toolbox performing ROI search, preprocessing, and false-positive filtering
    of MS1 data from multi-platform CE-MS/HILIC-MS runs
  repo: https://github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts vendor raw MS data to .mzXML or .mzML input format required by AriumMS
  repo: http://proteowizard.sourceforge.net/download.html
- name: ZMat toolbox
  role: Data compression toolbox bundled as a dependency of the AriumMS MATLAB application
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:3aa4aaa5b09ee3200c90ac3d3381fdeb1f58c85f67387b2f507b2ee58dad1056
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
  tools:
  - AriumMS
  - ZMat toolbox
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# roi-extraction-from-ms1-scans

## Summary

Extracts regions of interest (ROIs) from consecutive MS1 scans in mzXML/mzML metabolomics data as implemented in AriumMS, reducing computation time and false-positive features before peak detection in multi-platform CE-MS and HILIC-MS workflows.

## When to use

Use when raw CE-MS or LC-MS (mzXML/mzML) MS1 data must be reduced to candidate ROIs (m/z, scan-based region, intensity) prior to feature detection, especially for multi-platform metabolomics augmentation.

## When NOT to use

- Input is already a preprocessed feature table rather than raw mzXML/mzML MS1 data — ROI search would be redundant
- Data are MS/MS-level spectra or targeted MS2 acquisitions; AriumMS ROI search operates on MS1 scans and MS/MS spectral networking is not implemented
- Single-platform analysis where existing tools like XCMS Online, MetaboAnalyst, MetaboAnalystR, or MZmine already suffice

## Inputs

- Raw MS1 metabolomics data in .mzXML or .mzML format (CE-MS via nanoCEasy interface or HILIC-MS, ESI positive/negative)
- User-defined intensity threshold
- Minimum ROI size as number of consecutive MS1 scans
- Optional crop filter settings for retention-time/electropherogram window

## Outputs

- Preprocessed ROI table with m/z, scan-based region, and intensity after baseline correction
- ROI candidates surviving isotope, adduct, and contaminant false-positive filtering

## How to apply

Convert raw vendor files to .mzXML or .mzML with ProteoWizard msconvert, then run the AriumMS ROI search (ROIpeaks/MSroiaug functions from Tauler/Gorrochategui/Jaumot) with a user-defined intensity threshold and a minimum ROI size expressed as a number of consecutive MS1 scans. Apply the optional crop filter to restrict the retention-time/electropherogram window, discarding measurement areas without relevant peaks to cut computation time. Apply isotope, adduct, and common contaminant filters to minimize false-positive ROI candidates, then perform moving-window interpolation baseline correction during preprocessing. The output is a preprocessed ROI table (m/z, scan-based region, baseline-corrected intensity) that feeds AriumMS feature detection, integration, and mid-level data fusion augmentation of CE-MS and HILIC-MS datasets.

## Related tools

- **AriumMS** (Primary toolbox performing ROI search, preprocessing, and false-positive filtering of MS1 data from multi-platform CE-MS/HILIC-MS runs) — https://github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts vendor raw MS data to .mzXML or .mzML input format required by AriumMS) — http://proteowizard.sourceforge.net/download.html
- **ZMat toolbox** (Data compression toolbox bundled as a dependency of the AriumMS MATLAB application) — http://github.com/fangq/zmat

## Examples

```
msconvert raw.dta --mzXML -o converted/ # then open AriumMS app in MATLAB R2024a and run ROI search on the mzXML with intensity threshold, min consecutive scans, and crop filter
```

## Evaluation signals

- ROIs corresponding to known ground-truth peaks (e.g., in a synthetic mzXML with simulated ROI signals) survive filtering and baseline correction
- Output ROI table contains the expected columns: m/z, scan-based region, and baseline-corrected intensity
- ROI candidates flagged as isotopes, adducts, or known contaminants are removed by the false-positive filter stage
- Crop filter reduces computation time by discarding measurement areas without relevant peaks
- MATLAB R2024a or newer is used with the required toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database)

## Limitations

- The paper's raw CE-MS/LC-MS data are not publicly deposited, so validation may require synthetic mzXML inputs with simulated ground-truth ROIs
- MS/MS spectral networking is not implemented and is listed as a future development
- No version number is stated for AriumMS and no changelog is available
- Requires MATLAB R2024a or newer plus seven specific toolboxes for the app version

## Evidence

- [discussion] AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [readme] README: ROI search functions and input format: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [readme] README: input format conversion requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [readme] README: runtime requirements: "MATLAB R2024a or newer"
- [abstract] Abstract: ROI algorithm purpose across separation techniques: "AriumMS offers augmented data analysis of several separation techniques utilizing a region-of-interest algorithm"
