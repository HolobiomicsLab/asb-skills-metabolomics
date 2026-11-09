---
name: yeast-metabolite-target-counting
description: Use when comparing how many target metabolites each single separation
  method (HILIC-Orbitrap MS or CE-Orbitrap MS with nanoCEasy interface) detects in
  yeast extracts versus augmented multi-platform ESI-positive/ESI-negative data evaluations.
when_to_use_negative:
- The input is already an augmented or fused feature table — this skill requires counting
  from per-method processed ROI data before and after augmentation
- The goal is chemometric phenotype differentiation (e.g., PCA/clustering of inhibitor-treated
  vs. control yeast) rather than absolute coverage counting
- Data come from a single platform only — augmentation counts are meaningless without
  at least two separation methods per ionization mode
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Runs ROI search, preprocessing, feature detection and integration, false-positive
    filtering, and mid-level data fusion (augmentation) to produce per-method and
    augmented target/suspect counts
  repo: github.com/AdrianHaun/AriumMS
- name: ProteoWizard msconvert
  role: Converts vendor raw MS files to .mzXML or .mzML before loading into AriumMS
  repo: http://proteowizard.sourceforge.net/download.html
- name: fangq/zmat
  role: ZMat compression toolbox dependency used by AriumMS's ROIpeaks/MSroiaug functions
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:7bbeea92903df3b4939f950d55ef3a0c51d2ce05e559df29426f76425fb24c6d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ProteoWizard msconvert
  - fangq/zmat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# yeast-metabolite-target-counting

## Summary

Counts detected target and suspect metabolites in Saccharomyces cerevisiae extracts per analytical method (HILIC ESI+, HILIC ESI−, CE-MS methods) and after AriumMS mid-level augmentation of ESI-positive and ESI-negative platform sets, to quantify metabolome coverage gains from multi-platform fusion.

## When to use

Use when comparing how many target metabolites each single separation method (HILIC-Orbitrap MS or CE-Orbitrap MS with nanoCEasy interface) detects in yeast extracts versus augmented multi-platform ESI-positive/ESI-negative data evaluations.

## When NOT to use

- The input is already an augmented or fused feature table — this skill requires counting from per-method processed ROI data before and after augmentation
- The goal is chemometric phenotype differentiation (e.g., PCA/clustering of inhibitor-treated vs. control yeast) rather than absolute coverage counting
- Data come from a single platform only — augmentation counts are meaningless without at least two separation methods per ionization mode

## Inputs

- Yeast extract raw MS measurements (converted to .mzXML or .mzML via ProteoWizard msconvert) from five separation methods: three CE-Orbitrap MS (nanoCEasy interface) and two HILIC-Orbitrap MS methods
- Target and suspect metabolite lists for Saccharomyces cerevisiae
- Method-specific AriumMS processing parameters per separation method

## Outputs

- Per-method target and suspect counts (e.g., HILIC ESI+ 14 targets; CE cation 16 targets)
- ESI-negative augmentation counts: 24 targets / 38 suspects
- ESI-positive augmentation counts: 19 targets / 31 suspects
- Tabulated comparison of reproduced vs. reported reference values

## How to apply

Load the yeast extract raw MS measurements for the five separation methods (three CE-Orbitrap MS and two HILIC-Orbitrap MS) into AriumMS and run the full pipeline: ROI search, preprocessing, feature detection and integration, false-positive filtering, with method-specific parameters since the methods have different peak characteristics (CE-MS shows narrow peaks and benefits from nanoCEasy sensitivity). Record single-method target counts first, confirming reported values (HILIC ESI+ 14 targets; CE cation 16 targets). Then perform mid-level data fusion (augmentation) of the negative-ionization platforms (HILIC ESI− with CE alkaline BGE and CE acidic BGE) and count targets/suspects, and separately fuse the positive-ionization platforms (HILIC ESI+, CE cation) and count again. Tabulate all per-method and per-augmentation counts against reference values: 24 targets/38 suspects for ESI− augmentation and 19 targets/31 suspects for ESI+ augmentation; deviations indicate incorrect processing or false-positive filter settings.

## Related tools

- **AriumMS** (Runs ROI search, preprocessing, feature detection and integration, false-positive filtering, and mid-level data fusion (augmentation) to produce per-method and augmented target/suspect counts) — github.com/AdrianHaun/AriumMS
- **ProteoWizard msconvert** (Converts vendor raw MS files to .mzXML or .mzML before loading into AriumMS) — http://proteowizard.sourceforge.net/download.html
- **fangq/zmat** (ZMat compression toolbox dependency used by AriumMS's ROIpeaks/MSroiaug functions) — http://github.com/fangq/zmat

## Evaluation signals

- HILIC ESI+ single-method run yields exactly 14 target metabolites
- CE cation single-method run yields exactly 16 target metabolites
- ESI-negative augmentation (HILIC ESI− + CE alkaline BGE + CE acidic BGE) yields 24 targets and 38 suspects
- ESI-positive augmentation (HILIC ESI+ + CE cation) yields 19 targets and 31 suspects
- Augmented counts exceed all constituent single-method counts, confirming the expected coverage gain from mid-level data fusion

## Limitations

- Counts depend on method-specific processing and false-positive filtering parameters; incorrect settings produce deviations from the reported values (14, 16, 24/38, 19/31)
- MS/MS spectral networking is not implemented in AriumMS, so features detected across methods cannot be precisely and autonomously connected
- Requires MATLAB R2024a or newer plus Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes
- No version number stated for AriumMS; reproducibility across releases is not guaranteed

## Evidence

- [full_text] Single-method target counts: [evidence span withheld — anchor status: not_found]
- [discussion] Pipeline steps: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [abstract] Multi-platform scope: "To demonstrate the capabilities of AriumMS, five datasets were combined"
- [abstract] Flexible per-method parameters: "including parallel processing of datasets and flexible parameterization for processing of individual separation methods with different peak characteristics"
- [readme] File conversion requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project http://proteowizard.sourceforge.net/download.html"
- [discussion] Fusion rationale: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
