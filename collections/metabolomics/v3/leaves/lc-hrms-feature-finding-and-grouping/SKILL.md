---
name: lc-hrms-feature-finding-and-grouping
description: Use when raw LC-HRMS sample/blank/replicate analyses (vendor format or
  mzML) need automated feature extraction and cross-sample alignment — e.g. via patRoon
  with XCMS, OpenMS or enviPick — before filtering and annotation.
when_to_use_negative:
- A feature table with aligned feature groups already exists — proceed directly to
  patRoon's filter() cascade and annotation steps instead of re-running feature finding.
- You only need formula/compound annotation for already-detected features with MS/MS
  peak lists — this skill operates on raw LC-HRMS data, not annotation inputs.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: patRoon
  role: R platform exposing feature finding and grouping/alignment across analyses
    through one consistent interface, retaining per-feature MS/MS peak lists
  repo: https://github.com/rickhelmus/patRoon
- name: XCMS
  role: One of the three harmonized feature finding algorithms demonstrated in patRoon's
    benchmark comparison
- name: OpenMS
  role: Alternative harmonized feature finding algorithm selectable through the same
    patRoon interface
  repo: http://openms.de/
- name: enviPick
  role: Alternative harmonized feature finding algorithm for environmental LC-HRMS
    data
- name: ProteoWizard
  role: msconvert pre-treatment of raw vendor data to mzML before feature finding
    (used to prepare patRoonData)
  repo: http://proteowizard.sourceforge.net/index.shtml
- name: patRoonData
  role: Example triplicate standard-X / solvent-X LC-HRMS data package used to exercise
    the feature finding and grouping workflow
  repo: https://github.com/rickhelmus/patRoonData
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:2153f663d36f027005325984734012729e24255d7e1c5cdf50476775be2f7f92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - GenForm
  - MetFrag
  - OpenMS
  - ProteoWizard
  - R
  - SIRIUS
  - XCMS
  - enviPick
  - patRoon
  - patRoonData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# lc-hrms-feature-finding-and-grouping

## Summary

patRoon's harmonized LC-HRMS feature finding and grouping step: extract features from LC-HRMS runs with XCMS, OpenMS, enviPick or patRoon's native algorithms under one R interface, then align equivalent features across spiked samples, blanks and replicates while retaining per-feature MS/MS peak lists for downstream formula/compound annotation.

## When to use

Use when raw LC-HRMS sample/blank/replicate analyses (vendor format or mzML) need automated feature extraction and cross-sample alignment — e.g. via patRoon with XCMS, OpenMS or enviPick — before filtering and annotation.

## When NOT to use

- A feature table with aligned feature groups already exists — proceed directly to patRoon's filter() cascade and annotation steps instead of re-running feature finding.
- You only need formula/compound annotation for already-detected features with MS/MS peak lists — this skill operates on raw LC-HRMS data, not annotation inputs.

## Inputs

- Raw LC-HRMS analyses of environmental/spiked samples (vendor format or mzML)
- Triplicate blank-solvent analyses for blank subtraction (e.g. patRoonData 'solvent-X')
- Replicate analyses of a standard mixture (e.g. patRoonData 'standard-X', filtered to first 10 min and m/z 75-450)
- patRoonData example LC-HRMS data package for demonstrating the workflow

## Outputs

- Feature groups aligned across sample analyses (features with unique chromatographic and mass spectral properties grouped when equivalent)
- Automatically extracted and averaged MS/MS peak lists per feature
- Feature-intensity table across analyses, ready for patRoon's filter() cascade (blank, replicate, intensity)

## How to apply

Load the raw LC-HRMS analyses (spiked samples plus blank and replicate runs) into R through patRoon, pre-treating vendor data to mzML if needed (ProteoWizard msconvert, as used to prepare the patRoonData package). Run patRoon's feature finding with one of the harmonized algorithms (XCMS, OpenMS, enviPick, or patRoon's native ones), optionally preceded by automatic parameter optimization via IPO as the article suggests. Group/align features considered equivalent across all sample analyses so each unique chromatographic/mass spectral feature becomes one feature group spanning the spiked and blank analyses. Keep the automatically extracted and averaged MS/MS peak lists per feature group, because downstream formula (GenForm/SIRIUS) and compound (MetFrag) annotation scores candidates against those peaks. Expect a noisy result at this stage — the paper's demonstration yielded ~19000 feature groups that the subsequent filter() cascade (replicate, blank, intensity) cut to 4719/2933/964 — and use patRoon's consistent interface to benchmark algorithms by simply swapping the feature-finding algorithm without data-conversion overhead.

## Related tools

- **patRoon** (R platform exposing feature finding and grouping/alignment across analyses through one consistent interface, retaining per-feature MS/MS peak lists) — https://github.com/rickhelmus/patRoon
- **XCMS** (One of the three harmonized feature finding algorithms demonstrated in patRoon's benchmark comparison)
- **OpenMS** (Alternative harmonized feature finding algorithm selectable through the same patRoon interface) — http://openms.de/
- **enviPick** (Alternative harmonized feature finding algorithm for environmental LC-HRMS data)
- **ProteoWizard** (msconvert pre-treatment of raw vendor data to mzML before feature finding (used to prepare patRoonData)) — http://proteowizard.sourceforge.net/index.shtml
- **patRoonData** (Example triplicate standard-X / solvent-X LC-HRMS data package used to exercise the feature finding and grouping workflow) — https://github.com/rickhelmus/patRoonData

## Examples

```
remotes::install_github("rickhelmus/patRoon"); remotes::install_github("rickhelmus/patRoonData")
```

## Evaluation signals

- Equivalent features (matching chromatographic and mass spectral properties) are merged into one feature group spanning the spiked and blank analyses.
- Each retained feature group carries an automatically extracted/averaged MS/MS peak list, required by GenForm/SIRIUS formula and MetFrag compound annotation downstream.
- Feature-group count before filtering falls in the demonstrated range (~19000 groups in the spiked drinking-water demo, first replicate filter cut to 4719, -76%).
- Blank analyses (e.g. triplicate solvent blanks) are included in the grouping so the subsequent blank filter can operate.
- Swapping the feature-finding algorithm (XCMS <-> OpenMS <-> enviPick) through patRoon's interface yields comparable feature tables without manual data conversion.

## Limitations

- Different feature-finding algorithms produce differing results; the article notes existing tools 'overlap in functionality, and produce differing results', making algorithm choice a real decision point.
- Raw output is heavily redundant: ~19000 feature groups in the demo required rule-based filters that removed ~95%, so this stage alone is not interpretable without cleanup.
- Some integrated algorithms are closed-source ('connecting primarily open and a few closed approaches'), and algorithm/dependency versions are unspecified, deferring exact reproducibility to patRoon's installation instructions.

## Evidence

- [abstract] abstract: "This platform makes the use, evaluation and mixing of well-tested algorithms seamless by harmonizing various common (primarily open) software tools under a consistent interface"
- [intro] intro: [evidence span withheld — anchor status: not_found]
- [results] results: "The number of features for SIRIUS (except tests with native batch mode) and MetFrag benchmarks were further reduced by application of blank, replicate and intensity filters"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
