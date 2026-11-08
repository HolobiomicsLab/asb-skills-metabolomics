---
name: benchmark-result-reproduction
description: Use when a paper or task card reports a concrete quantitative result
  (feature counts, match rates, tolerance thresholds) that must be re-derived from
  the deposited datasets and workflow scripts to verify it reproduces.
when_to_use_negative:
- The input is a single dataset or raw .raw/.mzML files rather than two or more independently
  processed feature tables — use Gravity for redundant-feature clustering or Chroma
  for file reading instead
- Datasets were acquired on different LC-MS methods — Eclipse aligns 'same-method
  nontargeted LCMS datasets'
- The goal is drift correction or QC rather than cross-dataset alignment validation
  — use Blueshift or Formation instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3172
tools:
- name: Eclipse
  role: MSAligner performs the all-by-all DS1–DS4 alignment in default strict clique
    mode (diameter=1) whose output is benchmarked
  repo: github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA/IMDATA dictionaries) used to harmonize
    column headers across datasets and S1 scripts; installed via pip install bmxp
  repo: github.com/broadinstitute/bmxp
- name: Python
  role: Execution environment in which the datasets are merged, alignment is run,
    and counts are validated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:0573cd29fc63ca3ac47ac6fdb8aef78de062d53fc20b676d26c1e8d19a23dcd1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# benchmark-result-reproduction

## Summary

Reproduces a published benchmark result end-to-end (here: Eclipse's default strict clique-mode alignment of four human plasma LC-MS datasets, DS1–DS4) and validates the output against the paper's reported numbers (3861 combined features; 362/381 = 95% correctly matched overlapping annotations; 3 spurious annotation mismatches).

## When to use

Use when a paper or task card reports a concrete quantitative result (feature counts, match rates, tolerance thresholds) that must be re-derived from the deposited datasets and workflow scripts to verify it reproduces.

## When NOT to use

- The input is a single dataset or raw .raw/.mzML files rather than two or more independently processed feature tables — use Gravity for redundant-feature clustering or Chroma for file reading instead
- Datasets were acquired on different LC-MS methods — Eclipse aligns 'same-method nontargeted LCMS datasets'
- The goal is drift correction or QC rather than cross-dataset alignment validation — use Blueshift or Formation instead

## Inputs

- Four deposited human plasma LC-MS feature datasets (DS1–DS4), each with a Feature Metadata table
- Feature Abundances pivot tables per dataset (Compound_ID x injection_id)
- Supplementary Code S1 workflow scripts
- bmxp shared-schema dictionary objects (bmxp.FMDATA, bmxp.IMDATA)

## Outputs

- Aligned combined feature table across DS1–DS4 (expected 3861 features)
- Annotation-matching evaluation table counting correctly matched (362/381) and spurious (3) annotation-mismatch rows
- Validation report comparing reproduced counts to the published benchmark values

## How to apply

Load the deposited datasets (DS1–DS4, human plasma LC-MS feature tables) and merge each Feature Metadata table with its Feature Abundances pivot (Compound_ID x injection_id), since Eclipse requires merged Feature Metadata + Feature Abundances. Align the BMXP shared-schema column headers via the bmxp.FMDATA/IMDATA dictionary objects so the Supplementary Code S1 scripts and the datasets use consistent terminology (e.g. remap Compound_ID to Feature_ID if needed). Instantiate the Eclipse MSAligner from bmxp.eclipse and run it across all four datasets in default strict clique mode (diameter=1). Tabulate the aligned features and evaluate overlapping annotations across datasets, counting correctly matched rows and spurious annotation-mismatch rows. Validate the reproduction against the reported invariants: 3861 features (29% of the smallest dataset, DS2), 362 of 381 overlapping annotated features correctly matched (95%), and 3 spurious annotation-mismatch rows.

## Related tools

- **Eclipse** (MSAligner performs the all-by-all DS1–DS4 alignment in default strict clique mode (diameter=1) whose output is benchmarked) — github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (bmxp.FMDATA/IMDATA dictionaries) used to harmonize column headers across datasets and S1 scripts; installed via pip install bmxp) — github.com/broadinstitute/bmxp
- **Python** (Execution environment in which the datasets are merged, alignment is run, and counts are validated)

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner; aligner = MSAligner(diameter=1); aligned = aligner.align(ds1_merged, ds2_merged, ds3_merged, ds4_merged)
```

## Evaluation signals

- Aligned feature count equals 3861, i.e. 29% of the smallest dataset (DS2)
- Overlapping annotation match rate is 362/381 = 95% correctly matched
- Exactly 3 spurious annotation-mismatch rows appear in the evaluation table
- Each input dataset was merged Feature Metadata + Feature Abundances before alignment (Eclipse requirement)
- FMDATA/IMDATA dictionary remapping applied so S1 scripts and datasets share column terminology

## Limitations

- The ~16 s runtime reported for the benchmark is hardware-dependent and is treated as expert review, not a pass/fail criterion
- Eclipse handles matching scenarios for n > 2 datasets via a graph-based approach, but only for same-method nontargeted LCMS datasets
- All BMXP modules are standalone with no version numbers documented, and no full pipeline exists yet — steps must be run and validated individually

## Evidence

- [results] In default clique-mode alignment of DS1–DS4, Eclipse identified 3861 features (29% of the smallest dataset, DS2) and correctly matched 95% (362 of 381) of overlapping annotated features.: [evidence span withheld — anchor status: not_found]
- [methods] Eclipse requires merged Feature Metadata + Feature Abundances: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] MSAligner import and shared-schema remapping from the README: "These can be changed globally so that all packages will use the same terminology. To update the schema, modify the dictionary objects in the module directly prior to running code."
- [abstract] Graph-based approach scope statement from the abstract: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
