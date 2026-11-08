---
name: yeast-phenotype-regulation-calling
description: Use when AriumMS-processed augmented feature intensity tables (HILIC
  [ESI−]/[ESI+], CE alkaline BGE, CE acidic BGE, CE cation) from growth-inhibitor-treated
  vs. control Saccharomyces cerevisiae extracts must be reduced to per-target regulation
  calls and per-augmentation detection counts.
when_to_use_negative:
- Input is raw vendor MS data that has not yet passed AriumMS ROI search, preprocessing,
  feature detection/integration, false-positive filtering, and augmentation — run
  the AriumMS pipeline first
- The analysis is single-platform only (one LC-MS or CE-MS dataset); the paper notes
  that tools like XCMS Online, MetaboAnalyst, MetaboAnalystR, and MZmine focus on
  single-platform analysis and augmentation requires AriumMS
- A feature/annotation table with intensity values is needed rather than categorical
  fold-change regulation calls
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AriumMS
  role: Supplies the ROI search, preprocessing, feature detection and integration,
    false-positive filter, scaling, transformation, and mid-level augmentation that
    produce the fused feature tables on which fold-change regulation calls are computed
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts raw MS data to .mzXML or .mzML, the file format required upstream
    of AriumMS processing
  repo: http://proteowizard.sourceforge.net/download.html
- name: ZMat
  role: Compression toolbox dependency bundled by AriumMS (written by Qianqian Fang)
    used within the MATLAB app
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:2ccd22c1d096c48eb668fb0d7c0cf80fdd364468777e3f80cd8052184c0e5590
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ZMat
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# yeast-phenotype-regulation-calling

## Summary

Assigns per-metabolite fold-change regulation calls (down/stable/up/knock out/induced, with n.a. for undetected targets) for 40 yeast target metabolites from AriumMS mid-level-fused multi-platform CE-MS/HILIC-MS data, grouped into an ESI negative and an ESI positive augmentation.

## When to use

Use when AriumMS-processed augmented feature intensity tables (HILIC [ESI−]/[ESI+], CE alkaline BGE, CE acidic BGE, CE cation) from growth-inhibitor-treated vs. control Saccharomyces cerevisiae extracts must be reduced to per-target regulation calls and per-augmentation detection counts.

## When NOT to use

- Input is raw vendor MS data that has not yet passed AriumMS ROI search, preprocessing, feature detection/integration, false-positive filtering, and augmentation — run the AriumMS pipeline first
- The analysis is single-platform only (one LC-MS or CE-MS dataset); the paper notes that tools like XCMS Online, MetaboAnalyst, MetaboAnalystR, and MZmine focus on single-platform analysis and augmentation requires AriumMS
- A feature/annotation table with intensity values is needed rather than categorical fold-change regulation calls

## Inputs

- AriumMS mid-level-fused augmented feature intensity tables for the ESI negative augmentation group (HILIC [ESI−], CE alkaline BGE, CE acidic BGE)
- AriumMS mid-level-fused augmented feature intensity tables for the ESI positive augmentation group (HILIC [ESI+], CE cation)
- List of the 40 target metabolites tracked in the halogenated indole yeast experiment
- Sample-class metadata distinguishing growth-inhibitor-treated vs. control Saccharomyces cerevisiae extracts

## Outputs

- 40 × 2 regulation-call matrix (down/stable/up/knock out/induced/n.a.) for ESI negative and ESI positive augmentations
- Per-augmentation target detection counts (24 for ESI negative, 19 for ESI positive)
- table2_regulation_calls.csv containing the regulation-call matrix and augmentation totals

## How to apply

Load the AriumMS-processed augmented feature intensity tables for the two augmentation groups defined in the paper: the ESI negative group (HILIC ESI−, CE alkaline BGE, CE acidic BGE) and the ESI positive group (HILIC ESI+, CE cation). For each of the 40 target metabolites, compute the per-metabolite fold change between growth-inhibitor-treated and control yeast samples within each augmentation group, then assign one of the categorical calls down/stable/up/knock out/induced, using n.a. whenever a target was not detected in that augmentation group. Count detected targets per group and verify the expected totals of 24 targets for the anion (ESI negative) augmentation and 19 targets for the cation (ESI positive) augmentation before finalizing. Assemble the 40×2 regulation-call matrix plus the augmentation detection totals into one structured record and write it to table2_regulation_calls.csv. The rationale is that AriumMS's mid-level data fusion removes artifacts and noise across the LC-MS + CE-MS dataset, so regulation calls made on the fused tables are more robust than calls from any single platform.

## Related tools

- **AriumMS** (Supplies the ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, transformation, and mid-level augmentation that produce the fused feature tables on which fold-change regulation calls are computed) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts raw MS data to .mzXML or .mzML, the file format required upstream of AriumMS processing) — http://proteowizard.sourceforge.net/download.html
- **ZMat** (Compression toolbox dependency bundled by AriumMS (written by Qianqian Fang) used within the MATLAB app) — http://github.com/fangq/zmat

## Examples

```
msconvert --mzML yeast_hilic_neg.raw
```

## Evaluation signals

- ESI negative augmentation (HILIC [ESI−], CE alkaline BGE, CE acidic BGE) yields exactly 24 detected targets
- ESI positive augmentation (HILIC [ESI+], CE cation) yields exactly 19 detected targets
- Every one of the 40 target metabolites receives exactly one call per augmentation group, restricted to {down, stable, up, knock out, induced, n.a.}
- n.a. is assigned only to targets not detected in a given augmentation group (40 − 24 = 16 for ESI negative; 40 − 19 = 21 for ESI positive)
- Output file table2_regulation_calls.csv contains both the regulation-call matrix and the per-augmentation detection totals in a single structured record

## Limitations

- Targets absent from an augmentation group can only be labeled n.a., so regulation status is undetermined for 16 (ESI−) and 19 (ESI+) targets in this experiment
- MS/MS spectral networking to autonomously connect features across methods is not implemented in AriumMS and is listed as future development
- AriumMS requires MATLAB R2024a or newer with Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes; no version number is stated in the article

## Evidence

- [abstract] mid-level fusion output basis: "AriumMS provides a novel mid-level data fusion approach for multi-platform data analysis to simplify and speed up multi-platform data processing and evaluation"
- [discussion] AriumMS processing chain: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [discussion] mid-level fusion rationale for regulation calls: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [readme] upstream file-format requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project http://proteowizard.sourceforge.net/download.html"
