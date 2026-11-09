---
name: mid-level-data-fusion-evaluation
description: Use when combining untargeted MS datasets from different separation techniques
  (e.g., CE-MS and HILIC-MS in ESI+ and ESI−) into one augmented feature table to
  increase metabolome coverage beyond any single method.
when_to_use_negative:
- The analysis is single-platform only — tools like XCMS Online, MetaboAnalyst, MetaboAnalystR,
  or MZmine handle single-platform processing without augmentation
- The datasets cannot be processed with method-specific parameters because different
  separation techniques need individual settings (AriumMS exists precisely because
  standard tools do not support per-method preprocessing in one run)
- MS/MS spectral networking or coupling of spectroscopic data with chromatographic/electrophoretic
  data is required — these are not implemented in AriumMS
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: AriumMS
  role: Primary toolbox that runs ROI search, preprocessing, feature detection/integration,
    false-positive filtering, scaling, transformation, augmentation (mid-level data
    fusion), and chemometric evaluation; open-source MATLAB app at github.com/AdrianHaun/AriumMS
  repo: github.com/AdrianHaun/AriumMS
- name: ZMat
  role: Compression toolbox (zlib/gzip/lzma/lz4/zstd/blosc2 for MATLAB) used by AriumMS,
    written by Qianqian Fang
  repo: http://github.com/fangq/zmat
- name: msconvert
  role: ProteoWizard tool used to convert vendor raw MS data to .mzXML or .mzML before
    AriumMS processing
  repo: http://proteowizard.sourceforge.net/download.html
- name: nanoCEasy
  role: CE-MS interface used to acquire the three CE-Orbitrap MS datasets that feed
    the augmentation
- name: Orbitrap MS
  role: Mass spectrometer platform for all five input datasets (three CE-MS, two HILIC-MS
    methods)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:4251aa7f2a943f214d091ba423b5c038a75a8444b303214c0118919e3c7a7bb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - Orbitrap MS
  - ZMat
  - msconvert
  - nanoCEasy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mid-level-data-fusion-evaluation

## Summary

Performs mid-level data fusion (augmentation) of multi-platform untargeted metabolomics datasets — here three CE-Orbitrap MS and two HILIC-Orbitrap MS methods — using AriumMS's ROI-based workflow, yielding a single fused feature table for chemometric evaluation and target/suspect coverage counting.

## When to use

Use when combining untargeted MS datasets from different separation techniques (e.g., CE-MS and HILIC-MS in ESI+ and ESI−) into one augmented feature table to increase metabolome coverage beyond any single method.

## When NOT to use

- The analysis is single-platform only — tools like XCMS Online, MetaboAnalyst, MetaboAnalystR, or MZmine handle single-platform processing without augmentation
- The datasets cannot be processed with method-specific parameters because different separation techniques need individual settings (AriumMS exists precisely because standard tools do not support per-method preprocessing in one run)
- MS/MS spectral networking or coupling of spectroscopic data with chromatographic/electrophoretic data is required — these are not implemented in AriumMS

## Inputs

- Raw untargeted MS data for five separation methods: three CE-Orbitrap MS (cation, alkaline BGE, acidic BGE) and two HILIC-Orbitrap MS (ESI+, ESI−) runs of yeast (Saccharomyces cerevisiae) extracts
- MS files converted to .mzXML or .mzML format via ProteoWizard msconvert
- Target/suspect metabolite list for coverage counting

## Outputs

- Augmented (fused) feature table per ionization polarity from AriumMS mid-level data fusion
- Per-method and per-augmentation counts of detected target and suspect metabolites (reference: 14 HILIC ESI+; 16 CE cation; 24/38 ESI− augmentation; 19/31 ESI+ augmentation)
- Chemometric data-evaluation results from the augmented dataset

## How to apply

Convert raw MS data to .mzXML or .mzML with ProteoWizard msconvert, then load the five yeast-extract datasets (three CE-Orbitrap MS via the nanoCEasy interface, two HILIC-Orbitrap MS) into the AriumMS MATLAB app (R2024a+, with Bioinformatics, Statistics, Wavelet, Image Processing, Signal, Parallel Computing, and Database toolboxes). Run the pipeline per method in parallel with method-specific parameters — ROI search, preprocessing, feature detection and integration, false-positive filtering, scaling, and transformation — which AriumMS wraps around the ROIpeaks/MSroiaug functions of Tauler/Gorrochategui/Jaumot and the ZMat compression toolbox. Then perform mid-level data fusion separately per ionization polarity: augment HILIC ESI− with the CE alkaline and acidic BGE methods (expected 24 targets / 38 suspects) and HILIC ESI+ with CE cation (expected 19 targets / 31 suspects). Judge correctness by comparing per-method and augmented counts against references (HILIC ESI+ 14 targets; CE cation 16 targets) — deviations indicate incorrect processing or false-positive-filter settings.

## Related tools

- **AriumMS** (Primary toolbox that runs ROI search, preprocessing, feature detection/integration, false-positive filtering, scaling, transformation, augmentation (mid-level data fusion), and chemometric evaluation; open-source MATLAB app at github.com/AdrianHaun/AriumMS) — github.com/AdrianHaun/AriumMS
- **ZMat** (Compression toolbox (zlib/gzip/lzma/lz4/zstd/blosc2 for MATLAB) used by AriumMS, written by Qianqian Fang) — http://github.com/fangq/zmat
- **msconvert** (ProteoWizard tool used to convert vendor raw MS data to .mzXML or .mzML before AriumMS processing) — http://proteowizard.sourceforge.net/download.html
- **nanoCEasy** (CE-MS interface used to acquire the three CE-Orbitrap MS datasets that feed the augmentation)
- **Orbitrap MS** (Mass spectrometer platform for all five input datasets (three CE-MS, two HILIC-MS methods))

## Examples

```
msconvert yeast_raw.d --mzML  % then open AriumMS.mlapp in MATLAB R2024a, load the five mzML files, set per-method parameters, run ROIpeaks/MSroiaug augmentation
```

## Evaluation signals

- Single-method target counts match reported values: 14 targets for HILIC ESI+ and 16 targets for CE cation
- ESI− augmentation (HILIC ESI− + CE alkaline BGE + CE acidic BGE) yields 24 targets and 38 suspects
- ESI+ augmentation (HILIC ESI+ + CE cation) yields 19 targets and 31 suspects
- Augmented counts exceed every single-method count, confirming that fusion enhances coverage rather than losing features
- AriumMS mid-level fusion removes irrelevant artifacts and noise from the combined LC-MS + CE-MS yeast-extract dataset without dropping genuine phenotype-separating features

## Limitations

- No version number is stated for AriumMS; the README requires MATLAB R2024a or newer plus seven specific toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database)
- MS/MS spectral networking to connect features across two or more methods is not implemented and is listed as future development
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not implemented
- Existing tools (XCMS Online, MetaboAnalyst, MetaboAnalystR, MZmine) do not support augmentation of different separation techniques with per-dataset preprocessing settings, so manual augmentation or AriumMS is required

## Evidence

- [abstract] Mid-level fusion counts: "AriumMS provides a novel mid-level data fusion approach for multi-platform data analysis to simplify and speed up multi-platform data processing and evaluation"
- [discussion] Workflow composition: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [discussion] Fusion purpose: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [readme] AriumMS functions and dependencies: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [readme] File-format conversion requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [abstract] Study design / dataset composition: "To demonstrate the capabilities of AriumMS, five datasets were combined"
