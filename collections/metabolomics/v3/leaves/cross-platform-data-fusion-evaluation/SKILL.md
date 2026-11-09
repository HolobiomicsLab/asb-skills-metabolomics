---
name: cross-platform-data-fusion-evaluation
description: Use when multiple separation-platform MS feature tables (e.g., HILIC-MS
  and CE-MS in ESI positive and ESI negative modes) must be augmented/fused into one
  mid-level dataset to compare phenotypes, such as inhibitor-treated vs. control Saccharomyces
  cerevisiae extracts.
when_to_use_negative:
- Single-platform datasets (e.g., one HILIC-MS or one CE-MS run alone) — augmentation
  is designed for combining different separation techniques, and standard single-platform
  tools like XCMS Online, MetaboAnalyst, MetaboAnalystR or MZmine already cover that
  case
- Analyses requiring MS/MS spectral networking or linking features across methods
  by spectra — not implemented in AriumMS and listed only as future development
- Augmentation of spectroscopic data with chromatographic/electrophoretic data — mentioned
  as possible future work, not supported
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AriumMS
  role: Open-source MATLAB toolbox that performs ROI search, preprocessing, false-positive
    filtering, scaling/transformation, augmentation (mid-level data fusion) and chemometric
    evaluation of the multi-platform CE-MS/LC-MS data
  repo: github.com/AdrianHaun/AriumMS
- name: ZMat
  role: Compression toolbox dependency (MATLAB/Octave/Python) bundled by AriumMS for
    its data handling
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:45d1cea7a4f61f58b2444d6c1029c6ff5dfa29f1bc14b89bcf83942310971f98
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ZMat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-platform-data-fusion-evaluation

## Summary

Mid-level data fusion and chemometric evaluation of multi-platform untargeted metabolomics data (HILIC-MS ESI+/ESI− plus CE-MS) using AriumMS's ROI-based augmentation, yielding fold-change regulation calls for target metabolites across yeast phenotypes.

## When to use

Use when multiple separation-platform MS feature tables (e.g., HILIC-MS and CE-MS in ESI positive and ESI negative modes) must be augmented/fused into one mid-level dataset to compare phenotypes, such as inhibitor-treated vs. control Saccharomyces cerevisiae extracts.

## When NOT to use

- Single-platform datasets (e.g., one HILIC-MS or one CE-MS run alone) — augmentation is designed for combining different separation techniques, and standard single-platform tools like XCMS Online, MetaboAnalyst, MetaboAnalystR or MZmine already cover that case
- Analyses requiring MS/MS spectral networking or linking features across methods by spectra — not implemented in AriumMS and listed only as future development
- Augmentation of spectroscopic data with chromatographic/electrophoretic data — mentioned as possible future work, not supported

## Inputs

- Raw MS data converted to .mzXML or .mzML via ProteoWizard msconvert
- AriumMS-processed augmented feature intensity tables for the ESI negative group (HILIC ESI−, CE alkaline BGE, CE acidic BGE)
- AriumMS-processed augmented feature intensity tables for the ESI positive group (HILIC ESI+, CE cation)
- Sample metadata distinguishing growth-inhibitor-treated vs. control Saccharomyces cerevisiae extracts

## Outputs

- Augmented mid-level fused feature table per ion mode (ESI negative and ESI positive groups)
- Per-metabolite fold changes for the 40 target metabolites in each augmentation group
- Regulation-call matrix with calls down/stable/up/knock out/induced/n.a. per target per augmentation
- Structured record of regulation calls and augmentation totals written to table2_regulation_calls.csv

## How to apply

Convert raw vendor data to .mzXML or .mzML with ProteoWizard msconvert, then in AriumMS (MATLAB R2024a+, using the Tauler/Gorrochategui/Jaumot ROIpeaks and MSroiaug functions and the zmat compression toolbox) run ROI search, preprocessing, feature detection/integration, false-positive filtering, scaling and transformation for each platform separately. Group methods by ion mode for augmentation: the ESI negative group combines HILIC (ESI−), CE alkaline BGE and CE acidic BGE; the ESI positive group combines HILIC (ESI+) and CE cation. Mid-level augmentation removes artifacts and noise from the combined LC-MS + CE-MS dataset before chemometric evaluation. For the 40 target metabolites, compute per-metabolite fold changes between growth-inhibitor-treated and control yeast in each augmentation group and assign calls of down/stable/up/knock out/induced, using n.a. where a target was not detected in that group. Verify the augmentation detects 24 targets (ESI negative) and 19 targets (ESI positive) before assembling the final regulation-call matrix (e.g., table2_regulation_calls.csv).

## Related tools

- **AriumMS** (Open-source MATLAB toolbox that performs ROI search, preprocessing, false-positive filtering, scaling/transformation, augmentation (mid-level data fusion) and chemometric evaluation of the multi-platform CE-MS/LC-MS data) — github.com/AdrianHaun/AriumMS
- **ZMat** (Compression toolbox dependency (MATLAB/Octave/Python) bundled by AriumMS for its data handling) — http://github.com/fangq/zmat

## Evaluation signals

- Target detection totals match the reported values: 24 targets detected with the ESI negative augmentation and 19 with the ESI positive augmentation out of 40 targets
- Every one of the 40 target metabolites receives exactly one regulation call (down/stable/up/knock out/induced) or n.a. per augmentation group
- n.a. appears only where a target was genuinely not detected in that augmentation group, not as a placeholder for missing fold-change computation
- Mid-level fused dataset no longer carries artifacts and noise present in the individual LC-MS/CE-MS feature tables (the stated purpose of the fusion step)
- ESI negative group contains exactly three platforms (HILIC ESI−, CE alkaline BGE, CE acidic BGE) and the ESI positive group exactly two (HILIC ESI+, CE cation)

## Limitations

- MS/MS spectral networking to precisely and autonomously connect features across methods is not implemented; listed as future development
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not supported (possible future work)
- Requires MATLAB R2024a or newer plus multiple toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database) for the app version
- No version number or changelog is stated for AriumMS in the article

## Evidence

- [abstract] Mid-level fusion approach: "AriumMS provides a novel mid-level data fusion approach for multi-platform data analysis to simplify and speed up multi-platform data processing and evaluation"
- [discussion] AriumMS pipeline steps: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [discussion] Mid-level fusion purpose: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [intro] Multi-platform measurement design: "the metabolic standard mixtures and the yeast extracts were measured within a multi-platform approach combining HILIC-MS (ESI positive/ESI negative) with three CE-MS methods"
- [readme] Underlying ROI functions: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot https://doi.org/10.1038/protex.2015.102"
