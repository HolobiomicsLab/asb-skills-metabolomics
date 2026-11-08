---
name: lc-ms-feature-finding
description: Use when raw LC-HRMS data (e.g., a standard mixture vs. blank solvent,
  each in triplicate) must be converted to features grouped across analyses, especially
  to compare feature-finding algorithms or build a consensus feature list.
when_to_use_negative:
- The input is already a processed feature table — this skill covers raw-data-to-features
  extraction, not filtering or annotation of existing feature datasets
- A single-algorithm targeted workflow is needed with no interest in cross-algorithm
  comparison or consensus building
- Data is non-chromatographic direct-infusion or MS/MS-only; patRoon's feature finding
  assumes chromatographic LC (or GC) HRMS runs with retention-time separation
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: patRoon
  role: R platform providing the harmonized findFeatures/groupFeatures interface that
    runs and combines the feature-finding algorithms
  repo: https://github.com/rickhelmus/patRoon
- name: OpenMS
  role: Feature-finding algorithm invoked through patRoon for independent feature
    detection on the same MS dataset
  repo: http://openms.de/
- name: XCMS
  role: Feature-finding algorithm invoked through patRoon for independent feature
    detection on the same MS dataset
- name: enviPick
  role: Feature-finding algorithm invoked through patRoon for independent feature
    detection on the same MS dataset
- name: patRoonData
  role: R package supplying the example LC-HRMS triplicate data (standard-X / solvent-X)
    used to exercise the workflow
  repo: https://github.com/rickhelmus/patRoonData
- name: ProteoWizard
  role: msconvert pre-treatment of raw vendor data (e.g., the mz 75-450 / first-ten-minute
    trimming applied to patRoonData) before feature finding
  repo: http://proteowizard.sourceforge.net/index.shtml
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:56bd24475d4ffc396343aad27cb71808bcc3de9640ac8dece418328febf6976b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - OpenMS
  - ProteoWizard
  - R
  - XCMS
  - enviPick
  - patRoon
  - patRoonData
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# lc-ms-feature-finding

## Summary

Detect and group LC-HRMS features from environmental non-target data by running patRoon's harmonized feature-finding interface with multiple algorithms (OpenMS, XCMS, enviPick), then compare and combine the per-algorithm feature-group results.

## When to use

Use when raw LC-HRMS data (e.g., a standard mixture vs. blank solvent, each in triplicate) must be converted to features grouped across analyses, especially to compare feature-finding algorithms or build a consensus feature list.

## When NOT to use

- The input is already a processed feature table — this skill covers raw-data-to-features extraction, not filtering or annotation of existing feature datasets
- A single-algorithm targeted workflow is needed with no interest in cross-algorithm comparison or consensus building
- Data is non-chromatographic direct-infusion or MS/MS-only; patRoon's feature finding assumes chromatographic LC (or GC) HRMS runs with retention-time separation

## Inputs

- Raw LC-HRMS data converted to vendor-neutral format (e.g., mzML via ProteoWizard msconvert)
- patRoonData example data: 'standard-X' standard-mixture triplicate and 'solvent-X' blank-solvent triplicate
- Sample analysis metadata (analyses, blanks, replicate grouping) used by patRoon for feature grouping and blank/replicate filters
- Optional suspect list (e.g., from the NORMAN Suspect List Exchange) for downstream suspect screening on found features

## Outputs

- Per-algorithm feature-group results (feature objects with chromatographic and mass-spectral properties, grouped across triplicate analyses)
- Feature-group counts for OpenMS, XCMS and enviPick
- Pairwise overlap statistics between the three algorithms (unique/shared/fully-overlapping feature groups), assembled as feature_group_comparison.csv
- A combined consensus feature-group dataset usable for downstream filtering and annotation

## How to apply

Load the LC-HRMS sample data (e.g., patRoonData's 'standard-X' standard mixture and 'solvent-X' blank solvent, each injected in triplicate) and define sample analyses/blank relations in patRoon's analysis information table; optionally precede feature finding with automatic parameter optimization via IPO. Run feature finding independently for each algorithm — OpenMS, XCMS, and enviPick — through patRoon's harmonized findFeatures interface (algorithms can be swapped without data-format conversions), then group features considered equivalent across sample analyses with groupFeatures for each result. Count feature groups per algorithm and compute pairwise overlap statistics between the algorithm results; in the paper this yielded a consensus of 1243 feature groups (541/44% unique to one algorithm, 332/27% shared by two, 370/30% fully overlapping). Apply patRoon's rule-based pre-filters (minimum intensity, blank presence, replicate abundance) after feature finding if feature datasets need reduction, noting that a suspect such as aniline can be missed at too-high intensity thresholds. The rationale is that patRoon wraps established tools under one consistent interface so algorithms can be used, evaluated and mixed seamlessly for environmental NTA.

## Related tools

- **patRoon** (R platform providing the harmonized findFeatures/groupFeatures interface that runs and combines the feature-finding algorithms) — https://github.com/rickhelmus/patRoon
- **OpenMS** (Feature-finding algorithm invoked through patRoon for independent feature detection on the same MS dataset) — http://openms.de/
- **XCMS** (Feature-finding algorithm invoked through patRoon for independent feature detection on the same MS dataset)
- **enviPick** (Feature-finding algorithm invoked through patRoon for independent feature detection on the same MS dataset)
- **patRoonData** (R package supplying the example LC-HRMS triplicate data (standard-X / solvent-X) used to exercise the workflow) — https://github.com/rickhelmus/patRoonData
- **ProteoWizard** (msconvert pre-treatment of raw vendor data (e.g., the mz 75-450 / first-ten-minute trimming applied to patRoonData) before feature finding) — http://proteowizard.sourceforge.net/index.shtml

## Examples

```
remotes::install_github("rickhelmus/patRoonData"); features <- findFeatures(analysisInfo, "openms"); fGroups <- groupFeatures(features, algorithm="greedy")
```

## Evaluation signals

- All three algorithms (OpenMS, XCMS, enviPick) return non-empty, valid feature-group sets from the same dataset
- The comparison table has one row per algorithm with a numeric feature-group count and numeric overlap statistics against the other two algorithms, saved as feature_group_comparison.csv
- Overlap statistics partition into unique/shared/fully-overlapping classes; paper reference point is 1243 consensus feature groups split 541 (44%) / 332 (27%) / 370 (30%)
- Suspect screening on the feature data detects the expected suspects — 17 of 18 suspects detected in each algorithm's dataset in the paper
- Feature groups are consistent across triplicate injections, and blank (solvent-X) features can be removed by the blank-presence filter

## Limitations

- Algorithm results differ substantially (44% of consensus groups unique to one algorithm), so a single algorithm under-covers the feature space; combining is recommended
- The missing suspect (aniline) could only be detected when lowering the intensity threshold of the filter() function — default filters can drop real features
- Some integrated workflow algorithms are closed-source, limiting full open reproducibility; dependency/algorithm versions are unspecified and deferred to external installation instructions
- patRoonData files are reduced to the first ten minutes of the chromatographic run with m/z 75-450, so absolute feature counts on the demo data are lower than full-range runs

## Evidence

- [abstract] the ability to easily use, combine and evaluate different algorithms was demonstrated for three commonly used feature finding algorithms: "the ability to easily use, combine and evaluate different algorithms was demonstrated for three commonly used feature finding algorithms"
- [intro] Feature extraction and grouping across samples: [evidence span withheld — anchor status: not_found]
- [intro] patRoonData example data: "The example dataset contains LC-MS data for a standard mixture with known composition ('standard-X') and a blank solvent ('solvent-X')"
- [results] Feature data pre-treatment: "Pre-treatment of feature data consisted of removal of features with low intensity and lacking MS/MS data"
- [results] Aniline suspect recovery: "the missing suspect (aniline) could be detected when lowering the intensity threshold of the filter() function"
- [readme] patRoonData content: [evidence span withheld — anchor status: external_doc]
