---
name: targeted-metabolite-feature-matching
description: Use when you have augmented CE-MS/HILIC-MS feature intensity tables (treated
  vs. control Saccharomyces cerevisiae) and must assign down/stable/up/knock out/induced
  calls to a known list of target metabolites and count detections per augmentation
  group.
when_to_use_negative:
- Input is already a processed feature table needing only statistics — AriumMS's ROI
  search, false-positive filter, and augmentation pipeline is for untargeted multi-platform
  raw data
- Data are single-platform only — existing tools such as XCMS Online, MetaboAnalystR,
  or MZmine are designed for single-platform analysis and suffice
- Analysis goal is MS/MS spectral networking or spectroscopic-data augmentation —
  the article lists both as not yet implemented (future developments)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: AriumMS
  role: Performs ROI search, preprocessing, feature detection/integration, false-positive
    filtering, scaling, augmentation and chemometric evaluation of the multi-platform
    feature tables used for target matching
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts raw MS data to .mzXML or .mzML file format required as AriumMS input
  repo: http://proteowizard.sourceforge.net/download.html
- name: ZMat
  role: Compression toolbox used internally by AriumMS (MATLAB dependency)
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:cb6c859a6f0ae394e01ac141b644a9b0db55b76bdcc30e881f366d7bd0652f34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ZMat
  - msconvert (ProteoWizard)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# targeted-metabolite-feature-matching

## Summary

Matches features from AriumMS-processed multi-platform CE-MS and HILIC-MS data against a predefined list of 40 target metabolites in yeast extracts, producing per-metabolite fold-change regulation calls and per-augmentation detection counts. This connects untargeted multi-platform feature detection to a targeted, tabular reporting layer.

## When to use

Use when you have augmented CE-MS/HILIC-MS feature intensity tables (treated vs. control Saccharomyces cerevisiae) and must assign down/stable/up/knock out/induced calls to a known list of target metabolites and count detections per augmentation group.

## When NOT to use

- Input is already a processed feature table needing only statistics — AriumMS's ROI search, false-positive filter, and augmentation pipeline is for untargeted multi-platform raw data
- Data are single-platform only — existing tools such as XCMS Online, MetaboAnalystR, or MZmine are designed for single-platform analysis and suffice
- Analysis goal is MS/MS spectral networking or spectroscopic-data augmentation — the article lists both as not yet implemented (future developments)

## Inputs

- Raw multi-platform MS data converted to .mzXML or .mzML (HILIC-MS ESI+/- and three CE-Orbitrap MS methods via nanoCEasy interface)
- AriumMS-processed augmented feature intensity tables from the mid-level data fusion output
- List of 40 target metabolites to be matched
- Treated (growth-inhibitor) and control Saccharomyces cerevisiae extract sample groups

## Outputs

- Regulation-call matrix (40 target metabolites x ESI negative and ESI positive augmentation groups) with calls: down/stable/up/knock out/induced/n.a.
- Per-augmentation detection counts (expected: 24 ESI negative, 19 ESI positive)
- Structured CSV record, e.g. table2_regulation_calls.csv

## How to apply

Convert raw vendor MS files to mzML/mzXML with ProteoWizard msconvert, then process each platform with AriumMS using the ROI search (ROIpeaks/MSroiaug), preprocessing, feature detection and integration, false-positive filter, and scaling/transformation steps before augmentation. Load the mid-level fusion output feature intensity tables for the two augmentation groups: ESI negative (HILIC ESI-, CE alkaline BGE, CE acidic BGE) and ESI positive (HILIC ESI+, CE cation). For each of the 40 target metabolites, compute fold changes between growth-inhibitor-treated and control samples within each augmentation group and assign a regulation call (down/stable/up/knock out/induced); a target not detected in a group is marked n.a. Count detected targets per group and verify the expected totals of 24 for the anion augmentation and 19 for the cation augmentation before assembling the regulation-call matrix into a single structured record (table2_regulation_calls.csv).

## Related tools

- **AriumMS** (Performs ROI search, preprocessing, feature detection/integration, false-positive filtering, scaling, augmentation and chemometric evaluation of the multi-platform feature tables used for target matching) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts raw MS data to .mzXML or .mzML file format required as AriumMS input) — http://proteowizard.sourceforge.net/download.html
- **ZMat** (Compression toolbox used internally by AriumMS (MATLAB dependency)) — http://github.com/fangq/zmat

## Evaluation signals

- Detected-target counts per augmentation group equal 24 for ESI negative (HILIC ESI-, CE alkaline BGE, CE acidic BGE) and 19 for ESI positive (HILIC ESI+, CE cation)
- Every one of the 40 target metabolites has exactly one call from {down, stable, up, knock out, induced, n.a.} in each augmentation group
- All targets marked n.a. in an augmentation group are genuinely absent from that group's augmented feature intensity table (no fold change computable)
- Fold changes are computed per augmentation group (not pooled), consistent with the mid-level fusion grouping of methods
- Output matrix and totals serialize to a single structured record (table2_regulation_calls.csv) with consistent schema

## Limitations

- MS/MS spectral networking is not implemented, so precise autonomous connection of features detected by two or more methods is unavailable
- Targets undetected in an augmentation group can only be labeled n.a., reducing effective coverage per group
- AriumMS requires MATLAB R2024a or newer plus six toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database) for the app version
- No version number or changelog for AriumMS is stated, complicating reproducibility pinning

## Evidence

- [abstract] Saccharomyces cerevisiae (yeast) was treated with a growth inhibitor, and AriumMS successfully differentiated the metabolome based on the augmented multi-platform CE-MS and HILIC-MS investigation: "Saccharomyces cerevisiae (yeast) was treated with a growth inhibitor, and AriumMS successfully differentiated the metabolome based on the augmented"
- [discussion] AriumMS pipeline steps: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] mid-level data fusion removes artifacts and noise: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [readme] README: input format and dependency functions: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
