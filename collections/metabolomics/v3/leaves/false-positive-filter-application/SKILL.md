---
name: false-positive-filter-application
description: Use when raw multi-platform untargeted metabolomics data (CE-MS and HILIC-MS
  of yeast extracts, converted to .mzXML/.mzML) must be filtered of false-positive
  features after ROI search and feature detection, prior to augmentation.
when_to_use_negative:
- The goal is single-platform-only untargeted processing with generic tools (XCMS
  Online, MetaboAnalyst, MetaboAnalystR, MZmine), which focus on one platform and
  are not designed for augmentation of different separation techniques
- The input is already a filtered feature table and no ROI search, feature detection,
  or artifact removal is required
- The task is to link features detected by two or more methods via MS/MS spectral
  networking — this is not implemented in AriumMS and is listed as future development
edam_operation: http://edamontology.org/operation_2928
tools:
- name: AriumMS
  role: Hosts the false-positive filter within its pipeline (ROI search, preprocessing,
    feature detection and integration, false-positive filter, scaling, transformation,
    augmentation) with method-specific parameters
  repo: https://github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts raw Orbitrap MS files to .mzXML or .mzML format required as AriumMS
    input before filtering
  repo: http://proteowizard.sourceforge.net/download.html
- name: zmat
  role: ZMat compression toolbox used by AriumMS (functions ROIpeaks, MSroiaug context)
  repo: https://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:fb40e158b40caf74008202a1cfea565d3faa56d377f9770772d18a39ed6a5257
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
  tools:
  - AriumMS
  - msconvert (ProteoWizard)
  - zmat
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# false-positive-filter-application

## Summary

Application of the false-positive filter step within the AriumMS pipeline for untargeted metabolomics of multi-platform CE-Orbitrap MS and HILIC-Orbitrap MS data, removing spurious features (artifacts and noise) before mid-level data fusion (augmentation). Correct filtering is validated by reproducing the paper's per-method and augmented target/suspect counts.

## When to use

Use when raw multi-platform untargeted metabolomics data (CE-MS and HILIC-MS of yeast extracts, converted to .mzXML/.mzML) must be filtered of false-positive features after ROI search and feature detection, prior to augmentation.

## When NOT to use

- The goal is single-platform-only untargeted processing with generic tools (XCMS Online, MetaboAnalyst, MetaboAnalystR, MZmine), which focus on one platform and are not designed for augmentation of different separation techniques
- The input is already a filtered feature table and no ROI search, feature detection, or artifact removal is required
- The task is to link features detected by two or more methods via MS/MS spectral networking — this is not implemented in AriumMS and is listed as future development

## Inputs

- Raw MS measurements of yeast extracts and metabolite standard mixtures from three CE-Orbitrap MS methods and two HILIC-Orbitrap MS methods
- MS data converted to .mzXML or .mzML format (via msconvert/ProteoWizard)
- Method-specific AriumMS preprocessing parameter settings for ROI search, preprocessing, and feature detection/integration

## Outputs

- Filtered feature table (targets and suspects) per single analytical method, e.g. 14 targets for HILIC ESI+ and 16 targets for CE cation
- Filtered feature tables for augmented (mid-level fused) ESI− and ESI+ platform sets, expected at 24 targets/38 suspects and 19 targets/31 suspects respectively
- Cleaned dataset free of artifacts and noise, ready for scaling, transformation, augmentation, and chemometric evaluation

## How to apply

Run the false-positive filter as one stage of the AriumMS toolbox (MATLAB R2024a or newer) after ROI search, preprocessing, and feature detection/integration, and before scaling/transformation and augmentation. AriumMS processes the five separation methods (three CE-Orbitrap MS, two HILIC-Orbitrap MS) with method-specific parameters and parallel processing, so filter settings must be tuned per method to its different peak characteristics (CE-MS shows narrow peaks and benefits from nanoCEasy sensitivity). The filter removes irrelevant information such as artifacts and noise from the combined LC-MS + CE-MS dataset of yeast extracts, enabling mid-level data fusion. Correct filtering is judged against the reference counts: 14 targets (HILIC ESI+), 16 targets (CE cation), 24 targets/38 suspects for the ESI− augmentation (HILIC ESI− + CE alkaline BGE + CE acidic BGE), and 19 targets/31 suspects for the ESI+ augmentation (HILIC ESI+ + CE cation); deviations indicate incorrect processing or filtering settings.

## Related tools

- **AriumMS** (Hosts the false-positive filter within its pipeline (ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, transformation, augmentation) with method-specific parameters) — https://github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts raw Orbitrap MS files to .mzXML or .mzML format required as AriumMS input before filtering) — http://proteowizard.sourceforge.net/download.html
- **zmat** (ZMat compression toolbox used by AriumMS (functions ROIpeaks, MSroiaug context)) — https://github.com/fangq/zmat

## Examples

```
msconvert yeast_extract.raw --mzML
```

## Evaluation signals

- Reproduced single-method target counts match reported values: 14 targets for HILIC ESI+ and 16 targets for CE cation
- Reproduced augmented counts match reported values: 24 targets/38 suspects for the ESI− augmentation (HILIC ESI−, CE alkaline BGE, CE acidic BGE) and 19 targets/31 suspects for the ESI+ augmentation (HILIC ESI+, CE cation)
- Irrelevant artifacts and noise present in the raw combined LC-MS + CE-MS dataset are removed after the filter step
- Deviations from the reported reference counts indicate incorrect processing or filtering settings and require re-tuning of method-specific parameters

## Limitations

- No explicit numeric filter thresholds are published in the article or README; filter settings must be adjusted per separation method with AriumMS's flexible parameterization
- Deviating target/suspect counts versus the reported reference values (14, 16, 24/38, 19/31) signal wrong processing or filtering settings but give no direct diagnostic of which parameter failed
- MS/MS spectral networking to precisely and autonomously connect features detected by two or more methods is not implemented (future development)

## Evidence

- [discussion] AriumMS pipeline stages and filter: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] Mid-level fusion removes artifacts and noise: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [readme] Parameterized processing prerequisite: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation. It uses functions (ROIpeaks, MSroiaug) developed by"
- [abstract] Flexible per-method processing: "including parallel processing of datasets and flexible parameterization for processing of individual separation methods with different peak characteristics"
