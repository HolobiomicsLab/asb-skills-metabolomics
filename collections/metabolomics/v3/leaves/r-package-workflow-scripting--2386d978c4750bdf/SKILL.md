---
name: r-package-workflow-scripting
description: Use when LC/GC-HRMS data must be processed end-to-end in R — feature
  finding, grouping, filtering, annotation — and you want to run or compare several
  algorithms under one consistent programmatic interface.
when_to_use_negative:
- Input is already a processed feature table from a single tool and no cross-algorithm
  comparison or harmonized reprocessing is needed
- The workflow requires a single fixed commercial-vendor tool rather than patRoon's
  primarily open, vendor-agnostic algorithm set
- Data are not chromatography-coupled HRMS (e.g. direct-infusion spectra) and patRoon's
  feature finding/grouping model does not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: patRoon
  role: R package providing the harmonized scripting interface that drives feature
    finding, grouping, filtering and comparison across algorithms
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Example raw LC-HRMS data package ('standard-X' standard mixture and 'solvent-X'
    blank solvent triplicates) used to exercise the scripted workflow
  repo: https://github.com/rickhelmus/patRoonData
- name: OpenMS
  role: One of the three feature finding algorithms invoked through patRoon's harmonized
    interface
- name: XCMS
  role: One of the three feature finding algorithms invoked through patRoon's harmonized
    interface
- name: enviPick
  role: One of the three feature finding algorithms invoked through patRoon's harmonized
    interface
- name: ProteoWizard
  role: Used for raw data pre-treatment such as vendor-to-mzML format conversion before
    R-side processing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:da9b7fa8bc41ef757a65b91f4564d3a21f1faf8a9c11e71e52163c03e4691157
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
  - patRoon 2.0
  - patRoonData
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- r-workflow-scripting
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# r-package-workflow-scripting

## Summary

Script a complete non-target analysis (NTA) workflow as R code using the patRoon package, whose harmonized interface lets one R session drive, mix and compare multiple underlying algorithms (e.g. OpenMS, XCMS, enviPick for feature finding) without manual data conversion.

## When to use

Use when LC/GC-HRMS data must be processed end-to-end in R — feature finding, grouping, filtering, annotation — and you want to run or compare several algorithms under one consistent programmatic interface.

## When NOT to use

- Input is already a processed feature table from a single tool and no cross-algorithm comparison or harmonized reprocessing is needed
- The workflow requires a single fixed commercial-vendor tool rather than patRoon's primarily open, vendor-agnostic algorithm set
- Data are not chromatography-coupled HRMS (e.g. direct-infusion spectra) and patRoon's feature finding/grouping model does not apply

## Inputs

- LC/GC-HRMS raw data (vendor formats converted to mzML via ProteoWizard/OpenMS pre-treatment)
- Sample analysis metadata / analysis information table (e.g. patRoonData 'standard-X' and 'solvent-X' triplicates)
- Algorithm choice parameters (e.g. 'openms', 'xcms', 'envipick') and feature-finding settings
- Filtering thresholds (minimum intensity, blank presence, replicate abundance, RT and m/z ranges)

## Outputs

- Per-algorithm feature objects (featureGroups) for OpenMS, XCMS and enviPick
- Consensus/overlap statistics across algorithms (e.g. 1243 feature groups: 541 unique to one algorithm, 332 shared by two, 370 fully overlapped)
- Comparison table such as feature_group_comparison.csv with per-algorithm counts and pairwise overlaps
- Filtered feature datasets after blank, replicate and intensity filters

## How to apply

Install patRoon (and example data via remotes::install_github('rickhelmus/patRoonData') if needed), then drive the workflow from R scripts: load analyses such as the 'standard-X' standard mixture and 'solvent-X' blank solvent triplicates, call patRoon's harmonized feature-finding interface once per algorithm (OpenMS, XCMS, enviPick), and group features across sample analyses for each result. Compare algorithms by counting feature groups per tool and computing pairwise overlaps, assembling the result into a single comparison table (e.g. feature_group_comparison.csv). Refine feature datasets with patRoon's rule-based filter() function in the documented order — intensity pre-filter, retention time and m/z range filters, minimum replicate abundance, blank presence, minimum intensity, repetition of the replicate filter — because some filters depend on prior steps. The rationale is that patRoon harmonizes well-tested external tools behind one interface, so algorithm swapping and mixing are seamless and results are directly comparable within one R session.

## Related tools

- **patRoon** (R package providing the harmonized scripting interface that drives feature finding, grouping, filtering and comparison across algorithms) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Example raw LC-HRMS data package ('standard-X' standard mixture and 'solvent-X' blank solvent triplicates) used to exercise the scripted workflow) — https://github.com/rickhelmus/patRoonData
- **OpenMS** (One of the three feature finding algorithms invoked through patRoon's harmonized interface)
- **XCMS** (One of the three feature finding algorithms invoked through patRoon's harmonized interface)
- **enviPick** (One of the three feature finding algorithms invoked through patRoon's harmonized interface)
- **ProteoWizard** (Used for raw data pre-treatment such as vendor-to-mzML format conversion before R-side processing)

## Examples

```
remotes::install_github("rickhelmus/patRoonData"); library(patRoon); fDataOM <- findFeatures(anaInfo, algorithm = "openms")
```

## Evaluation signals

- All three algorithms (OpenMS, XCMS, enviPick) produce valid feature-group sets from the same MS dataset
- Comparison table contains one row per algorithm with numeric feature-group counts and numeric pairwise overlap statistics against the other two algorithms
- Overlap statistics are complete and consistent with a three-way partition (unique to one / shared by two / fully overlapped summing to the total feature groups)
- Suspect screening consistency: the same suspects (17 of 18, with aniline recoverable only by lowering the filter() intensity threshold) are detected across per-algorithm feature datasets
- Filter application order is respected (intensity pre-filter → RT/m/z range → replicate/blank/intensity → replicate filter repetition) and filtered counts decrease monotonically

## Limitations

- patRoon version, algorithm dependency versions and installation requirements are not stated in the article and are deferred to external installation instructions
- Some integrated algorithms are closed-source, limiting full open reproducibility
- Benchmark/demonstration source code is only available as supplementary archive files (e.g. Additional file 4), not in the main repository text
- Overly strict filtering thresholds can hide real compounds — the aniline suspect was missed until the filter() intensity threshold was lowered

## Evidence

- [abstract] This platform makes the use, evaluation and mixing of well-tested algorithms seamless by harmonizing various common (primarily open) software tools under a consistent interface: "This platform makes the use, evaluation and mixing of well-tested algorithms seamless by harmonizing various common (primarily open) software tools under a consistent interface"
- [intro] Example data is contained in the patRoonData R package: "Example data is contained in the patRoonData R package"
- [readme] Feature extraction | Finding features and grouping them across analyses. | Native (piek, greedy), XCMS, OpenMS, enviPick: [evidence span withheld — anchor status: external_doc]
- [results] the missing suspect (aniline) could be detected when lowering the intensity threshold of the filter() function: "the missing suspect (aniline) could be detected when lowering the intensity threshold of the filter() function"
- [supplementary] Source code for benchmarks. Archive with several R scripts that were used to perform the parallelization benchmarks.: "Archive with several R scripts that were used to perform the parallelization benchmarks"
