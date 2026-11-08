---
name: isotope-adduct-contaminant-false-positive-filtering
description: Use when processing raw CE-MS or LC-MS (mzXML/mzML) metabolomics data
  with the AriumMS ROI workflow and you need to discard isotope/adduct/contaminant-derived
  ROI candidates that would inflate the feature list before data augmentation.
when_to_use_negative:
- Input is already a curated, annotated feature table (the filter operates on ROI
  candidates during ROI search/preprocessing, not post-annotation tables)
- Single-platform targeted analysis where existing tools like XCMS Online, MetaboAnalyst,
  MetaboAnalystR, or MZmine are adequate and no augmentation of separation techniques
  is needed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Toolbox implementing the false-positive filter stage within its ROI search,
    preprocessing, and augmentation pipeline
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert
  role: ProteoWizard tool used to convert raw MS data to .mzXML/.mzML before ROI search
    and filtering
  repo: http://proteowizard.sourceforge.net/download.html
- name: ZMat
  role: MATLAB/Octave compression toolbox used as a dependency by AriumMS
  repo: https://github.com/NeuroJSON/zmat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:f115daca55ed273fed557d5b00dac8ec305e7751f9e0de26010d5611e609e977
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ZMat
  - msconvert
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# isotope-adduct-contaminant-false-positive-filtering

## Summary

A false-positive filtering stage in the AriumMS untargeted metabolomics pipeline that removes ROI candidates attributable to isotopes, adducts, and common contaminants before feature integration, reducing spurious features in multi-platform CE-MS/HILIC-MS data.

## When to use

Use when processing raw CE-MS or LC-MS (mzXML/mzML) metabolomics data with the AriumMS ROI workflow and you need to discard isotope/adduct/contaminant-derived ROI candidates that would inflate the feature list before data augmentation.

## When NOT to use

- Input is already a curated, annotated feature table (the filter operates on ROI candidates during ROI search/preprocessing, not post-annotation tables)
- Single-platform targeted analysis where existing tools like XCMS Online, MetaboAnalyst, MetaboAnalystR, or MZmine are adequate and no augmentation of separation techniques is needed

## Inputs

- ROI candidate table from AriumMS ROI search (m/z, scan-based region, intensity) for CE-MS and HILIC-MS datasets
- mzXML or mzML files converted with msconvert
- AriumMS processing parameters: intensity threshold, minimum ROI size (consecutive MS1 scans), optional crop window, filter toggles for isotope/adduct/contaminant removal

## Outputs

- Filtered ROI/feature table with isotope, adduct, and contaminant candidates removed, feeding feature detection, integration, scaling, transformation, and augmentation

## How to apply

First convert vendor MS files with ProteoWizard msconvert to .mzXML or .mzML. Run the AriumMS ROI search (user-defined intensity threshold, minimum ROI size in consecutive MS1 scans, optional crop filter) using the ROIpeaks/MSroiaug functions, which yields ROI candidates across the parallel CE-Orbitrap MS and HILIC-Orbitrap MS datasets. Then enable the false-positive filter stage so ROI candidates flagged as isotopes, adducts, and known common contaminants are removed before feature detection, integration, and mid-level data fusion. Rationale: the filter minimizes false-positive features and, as part of the broader pipeline, removes irrelevant information such as artifacts and noise from the combined LC-MS + CE-MS dataset. Configure parameters per separation method, since flexible parameterization is offered for methods with different peak characteristics (narrow CE peaks vs. wider HILIC peaks).

## Related tools

- **AriumMS** (Toolbox implementing the false-positive filter stage within its ROI search, preprocessing, and augmentation pipeline) — github.com/AdrianHaun/AriumMS
- **msconvert** (ProteoWizard tool used to convert raw MS data to .mzXML/.mzML before ROI search and filtering) — http://proteowizard.sourceforge.net/download.html
- **ZMat** (MATLAB/Octave compression toolbox used as a dependency by AriumMS) — https://github.com/NeuroJSON/zmat

## Examples

```
msconvert yeast_extract.raw --mzML  # then open the AriumMS MATLAB app (R2024a+), set intensity threshold and min ROI size, and enable the isotope/adduct/contaminant false-positive filter before ROI search
```

## Evaluation signals

- ROI candidates corresponding to known ground-truth metabolite peaks (e.g., metabolic standard mixtures) survive the isotope/adduct/contaminant filters
- Feature list size drops relative to the unfiltered ROI table, with no loss of ground-truth standard features
- Downstream mid-level fusion removes irrelevant artifacts and noise from the combined LC-MS + CE-MS yeast-extract dataset
- AriumMS successfully differentiates metabolome groups (e.g., inhibited vs. control Saccharomyces cerevisiae) after filtering, indicating true features were retained
- All ROIs in the output carry post-baseline-correction intensities and pass the preceding ROI-size (consecutive MS1 scans) criterion

## Limitations

- MS/MS spectral networking for connecting features across methods is not implemented (listed as future development), so filter decisions rely on isotope/adduct/contaminant rules alone
- No version number or changelog is stated for AriumMS; behavior may vary with the MATLAB version (R2024a or newer required) and toolbox availability
- Paper's raw CE-MS/LC-MS data are not publicly deposited, so filter behavior must be validated on synthetic or in-house data

## Evidence

- [discussion] Pipeline stage list supporting filter placement: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] Filter removes artifacts/noise from fused data: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [readme] AriumMS ROI search uses ROIpeaks/MSroiaug with parameter-driven preprocessing: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation. It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [readme] msconvert required for input format conversion: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [intro] Filter minimizes false positives as part of toolbox purpose: "AriumMS offers automated batch processing with flexible processing options and a graphical user"
