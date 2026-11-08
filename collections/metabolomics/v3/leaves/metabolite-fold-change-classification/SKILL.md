---
name: metabolite-fold-change-classification
description: Use when AriumMS mid-level fused feature intensity tables for growth-inhibitor-treated
  vs. control Saccharomyces cerevisiae extracts are available and per-metabolite regulation
  calls must be reported per ESI augmentation group.
when_to_use_negative:
- Input is a single-platform (one HILIC or one CE method only) feature table rather
  than the augmented mid-level fused output from the ESI negative or ESI positive
  platform groups
- Samples are not paired treated-vs-reference yeast extracts (no growth-inhibitor
  treatment contrast), so fold-change calls are not meaningful
- The metabolite of interest is not among the 40 target metabolites; targets absent
  from an augmentation group must be recorded as n.a., not inferred from other platforms
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: AriumMS
  role: Produces the mid-level fused augmented feature intensity tables (ROI search,
    preprocessing, feature detection, false-positive filter, scaling, augmentation)
    from which per-target fold changes and regulation calls are computed
  repo: github.com/AdrianHaun/AriumMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:7fa6c1cb060785b15d2655ea77482fbfae6a74a1af300a434278a5b4a70bc6d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# metabolite-fold-change-classification

## Summary

Classifies each target metabolite in AriumMS-processed augmented multi-platform (HILIC-MS + CE-MS) yeast data into fold-change regulation calls (down/stable/up/knock out/induced, or n.a. if undetected), computed per augmentation group (ESI negative vs ESI positive).

## When to use

Use when AriumMS mid-level fused feature intensity tables for growth-inhibitor-treated vs. control Saccharomyces cerevisiae extracts are available and per-metabolite regulation calls must be reported per ESI augmentation group.

## When NOT to use

- Input is a single-platform (one HILIC or one CE method only) feature table rather than the augmented mid-level fused output from the ESI negative or ESI positive platform groups
- Samples are not paired treated-vs-reference yeast extracts (no growth-inhibitor treatment contrast), so fold-change calls are not meaningful
- The metabolite of interest is not among the 40 target metabolites; targets absent from an augmentation group must be recorded as n.a., not inferred from other platforms

## Inputs

- AriumMS mid-level fused feature intensity table for the ESI negative augmentation (HILIC [ESI-] + CE alkaline BGE + CE acidic BGE)
- AriumMS mid-level fused feature intensity table for the ESI positive augmentation (HILIC [ESI+] + CE cation)
- Sample metadata distinguishing growth-inhibitor-treated vs. control Saccharomyces cerevisiae extracts
- List of the 40 target metabolites with identities

## Outputs

- Regulation-call matrix (40 target metabolites x augmentation groups) with calls down/stable/up/knock out/induced/n.a.
- Per-augmentation detection totals (expected: 24 targets for ESI negative, 19 for ESI positive)
- Structured record written to table2_regulation_calls.csv

## How to apply

Load the AriumMS-processed augmented feature intensity tables for the ESI negative augmentation (HILIC [ESI-], CE alkaline BGE, CE acidic BGE) and the ESI positive augmentation (HILIC [ESI+, CE cation]) from the upstream mid-level fusion output. For each of the 40 target metabolites, compute the fold change between growth-inhibitor-treated and reference/control yeast samples within each augmentation group, keeping the two ionization-mode augmentations separate rather than pooling platforms. Assign one regulation call per target per group — down, stable, up, knock out, or induced — and use n.a. wherever a target was not detected in that augmentation's fused data. Finally count detected targets per group and verify the expected totals of 24 for the ESI negative augmentation and 19 for the ESI positive augmentation before assembling the regulation-call matrix.

## Related tools

- **AriumMS** (Produces the mid-level fused augmented feature intensity tables (ROI search, preprocessing, feature detection, false-positive filter, scaling, augmentation) from which per-target fold changes and regulation calls are computed) — github.com/AdrianHaun/AriumMS

## Evaluation signals

- Total detected targets equals 24 for the ESI negative augmentation (HILIC [ESI-], CE alkaline BGE, CE acidic BGE)
- Total detected targets equals 19 for the ESI positive augmentation (HILIC [ESI+, CE cation)]
- Every one of the 40 targets receives exactly one call per augmentation group, with n.a. used only where the target was not detected in that group
- Every call is one of the six allowed labels: down, stable, up, knock out, induced, n.a.
- Fold changes are computed per augmentation group separately (no cross-ionization-mode pooling) and the matrix is serialized to table2_regulation_calls.csv

## Limitations

- Targets not detected by a given augmentation group cannot be classified and must be recorded as n.a., so per-group call matrices are incomplete by design
- MS/MS spectral networking is not implemented in AriumMS and is listed only as a future development, so cross-method feature connection relies on the augmentation step
- No version number, changelog, or explicit code/license availability statement is given in the article for AriumMS; the README requires MATLAB R2024a or newer plus several toolboxes

## Evidence

- [abstract] AriumMS successfully differentiated the metabolome of Saccharomyces cerevisiae treated with a growth inhibitor using multi-platform CE-MS and HILIC-MS: "Saccharomyces cerevisiae (yeast) was treated with a growth inhibitor, and AriumMS successfully differentiated the metabolome based on the augmented multi-platform CE-MS and HILIC-MS investigation"
- [discussion] AriumMS mid-level fusion and processing pipeline that supplies the augmented intensity tables: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [discussion] AriumMS mid-level fusion removes artifacts and noise from yeast LC-MS + CE-MS data before evaluation: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [readme] README describing AriumMS ROI/preprocessing/augmentation pipeline and its DOI: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation. It uses functions (ROIpeaks, MSroiaug) developed by"
- [abstract] Augmentation grouping: five datasets combined across three CE-Orbitrap MS and two HILIC-Orbitrap MS methods: [evidence span withheld — anchor status: not_found]
