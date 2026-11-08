---
name: molecular-formula-assignment-from-accurate-mass
description: Use when LC-HRMS feature lists with accurate m/z and MS/MS peak lists
  exist and each feature needs ranked chemical formula candidates before compound
  annotation in patRoon.
when_to_use_negative:
- Features lack MS/MS spectra and only accurate mass is available — the demonstrated
  pre-treatment removed features with low intensity and lacking MS/MS data, so formula
  ranking quality degrades
- The task is full suspect screening of known compounds rather than formula assignment
  for unknown features in patRoon's NTA workflow
- The input is already an annotated compound table; then compound annotation/ranking
  (MetFrag) rather than formula assignment is the next step
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: patRoon
  role: R platform harmonizing the formula-annotation step under a consistent interface
    and exporting annotations to CSV
  repo: https://github.com/rickhelmus/patRoon
- name: GenForm
  role: Formula calculation backend from accurate mass and MS/MS spectra
- name: SIRIUS
  role: Alternative formula calculation backend; benchmarked with native batch mode
    and reduced feature sets
- name: MetFrag
  role: Consumes formula annotations to rank compound candidates from a local compound
    database against MS/MS peaks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:b1636c373e152aa6847f07c16b056b573a6f252768b179255222bffd9c94abd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - GenForm
  - MetFrag
  - OpenMS
  - R
  - SIRIUS
  - XCMS
  - patRoon
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# molecular-formula-assignment-from-accurate-mass

## Summary

Automatically assigns chemical formula candidates to HRMS features using accurate mass and MS/MS spectra via patRoon's GenForm or SIRIUS backends, producing ranked formulae that feed downstream compound annotation with MetFrag. This is the formula-annotation stage of patRoon's non-target analysis workflow demonstrated on spiked drinking-water treatment plant samples.

## When to use

Use when LC-HRMS feature lists with accurate m/z and MS/MS peak lists exist and each feature needs ranked chemical formula candidates before compound annotation in patRoon.

## When NOT to use

- Features lack MS/MS spectra and only accurate mass is available — the demonstrated pre-treatment removed features with low intensity and lacking MS/MS data, so formula ranking quality degrades
- The task is full suspect screening of known compounds rather than formula assignment for unknown features in patRoon's NTA workflow
- The input is already an annotated compound table; then compound annotation/ranking (MetFrag) rather than formula assignment is the next step

## Inputs

- HRMS feature list with accurate m/z values from LC-HRMS analyses (e.g. mzML-derived feature groups)
- MS/MS peak lists extracted per feature
- patRoon feature data surviving blank, replicate and intensity filters
- Local compound database files (e.g. CompTox CSV, PubChem cache) for downstream scoring

## Outputs

- Per-feature ranked chemical formula candidates
- feature_annotations.csv containing formula, compound candidates and rankings
- Formula assignments filtered to features with MS/MS data and sufficient intensity

## How to apply

After feature finding and clean-up filters in patRoon, calculate formula candidates for each feature with GenForm or SIRIUS using the feature's accurate mass and its MS/MS spectra. The demonstrated benchmark workflow first pre-treated the feature data by removing features with low intensity and lacking MS/MS data, then applied blank, replicate and intensity filters, which reduced feature groups from ~19000 to 964 (-95%), so only features surviving those filters and retaining MS/MS peaks enter formula calculation. In the patRoonData demonstration the correct formulae were ranked 1st for 13 of 16 candidates and within the top 5 for 100% of compounds, showing ranking quality depends on both accurate mass and MS/MS spectral information. Export the resulting per-feature formula assignments (e.g. via patRoon's annotation outputs) to a feature_annotations.csv for use in compound annotation and ranking with MetFrag, whose compound database (e.g. CompTox via a local CSV file) can be scored against MS/MS peaks. Note that the article benchmark used 'GenForm and SIRIUS' as the formula backends harmonized under patRoon's consistent interface.

## Related tools

- **patRoon** (R platform harmonizing the formula-annotation step under a consistent interface and exporting annotations to CSV) — https://github.com/rickhelmus/patRoon
- **GenForm** (Formula calculation backend from accurate mass and MS/MS spectra)
- **SIRIUS** (Alternative formula calculation backend; benchmarked with native batch mode and reduced feature sets)
- **MetFrag** (Consumes formula annotations to rank compound candidates from a local compound database against MS/MS peaks)

## Examples

```
R> # within patRoon after feature filtering: features <- filter(...); fdata <- calculateFeatures(formulas = "genform") ; annotations <- patRoon::generateCompounds(formulas = fdata)
```

## Evaluation signals

- Correct formula ranked 1st for 13 of 16 candidates in the patRoonData benchmark
- Correct formula within top 5 for 100% of compounds in the benchmark
- Feature count matches expected reduction: ~19000 → 4719 (replicate) → 2933 (blank) → 964 (intensity) before annotation
- Output feature_annotations.csv contains per-feature formula and compound candidate/ranking columns
- Only features retaining MS/MS peak lists and surviving the blank/replicate/intensity filters are annotated

## Limitations

- Features lacking MS/MS data or with low intensity are removed in pre-treatment, so they cannot be annotated
- SIRIUS tests with native batch mode were excluded from the benchmark reduction pathway
- patRoon harmonizes a few closed-source algorithms, so some backends limit full open reproducibility
- Aniline suspect was missed at the demonstrated intensity threshold; lowering the filter threshold recovers it
- Repository lacks a stated patRoon software version and changelog for the snapshot

## Evidence

- [results] patRoon formula annotation benchmark with GenForm/SIRIUS: [evidence span withheld — anchor status: not_found]
- [results] Formula candidates ranked correctly: [evidence span withheld — anchor status: not_found]
- [results] Filter pipeline before annotation: [evidence span withheld — anchor status: not_found]
- [abstract] patRoon harmonized interface rationale: "This platform makes the use, evaluation and mixing of well-tested algorithms seamless by harmonizing various common (primarily open) software tools under a consistent interface"
- [intro] patRoon scope including formula annotation: [evidence span withheld — anchor status: not_found]
- [results] Pre-treatment removal criteria: "Pre-treatment of feature data consisted of removal of features with low intensity and lacking MS/MS data"
- [readme] GenForm and SIRIUS as formula backends: [evidence span withheld — anchor status: external_doc]
- [results] MetFrag consumption of formula annotations: "the compound database used by MetFrag (here CompTox via a local CSV file obtained from [119])"
