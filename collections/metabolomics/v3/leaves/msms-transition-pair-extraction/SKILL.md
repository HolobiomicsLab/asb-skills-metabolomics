---
name: msms-transition-pair-extraction
description: Use when a curated tandem MS library (msp format, e.g. from mspcompiler
  output) must be converted into pseudo-MRM transition tables for MRMPROBS so chemicals
  already identified in DIA data can be sensitively re-screened without re-acquisition.
when_to_use_negative:
- When the goal is initial untargeted annotation of unknown features in the DIA data
  (this skill presupposes compounds have already been identified and curated in the
  library).
- When the library entries contain fewer than 5 product ions per compound (the top-5-per-compound
  rule cannot be satisfied; fewer transitions would result, breaking the validation
  invariant).
- When data were acquired in true MRM mode on a triple quadrupole — MRMPROBS pseudo-MRM
  retrospective targeting applies to DIA data sets already acquired.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mspcompiler
  role: produces the curated, verified msp MS/MS library whose entries are parsed
    for precursor and product-ion records
  repo: https://github.com/QizhiSu/mspcompiler
- name: MRMPROBS
  role: targeted-metabolomics software that loads the pseudo-MRM transition table
    and provides automated posterior probabilistic data assessment on DIA data
- name: MS-DIAL
  role: processed the DIA data (version 4.38) that the pseudo-MRM transitions are
    applied to
- name: R
  role: environment in which mspcompiler and the transition-pair extraction logic
    run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:c8584860c28e634d8c99009e96e8907905883e93531436ead54b406edba0dfbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - MRMPROBS
  - MS-DIAL
  - R
  - mspcompiler
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# msms-transition-pair-extraction

## Summary

Convert curated in-house MS/MS library entries (449 positive-mode and 172 negative-mode plastic-related chemicals in verified msp files) into MRMPROBS pseudo-MRM transition tables by parsing each compound's precursor m/z and ranking its product ions by intensity, keeping the top 5 as precursor-product (Q1→Q3) pairs for targeted screening of DIA-acquired recycled HDPE data.

## When to use

Use when a curated tandem MS library (msp format, e.g. from mspcompiler output) must be converted into pseudo-MRM transition tables for MRMPROBS so chemicals already identified in DIA data can be sensitively re-screened without re-acquisition.

## When NOT to use

- When the goal is initial untargeted annotation of unknown features in the DIA data (this skill presupposes compounds have already been identified and curated in the library).
- When the library entries contain fewer than 5 product ions per compound (the top-5-per-compound rule cannot be satisfied; fewer transitions would result, breaking the validation invariant).
- When data were acquired in true MRM mode on a triple quadrupole — MRMPROBS pseudo-MRM retrospective targeting applies to DIA data sets already acquired.

## Inputs

- curated in-house MS/MS library in msp format (compiled by mspcompiler via write_MS2_msp, e.g. combine_ms2_pos.msp / combine_ms2_neg.msp), containing 449 positive-mode and 172 negative-mode plastic-related chemical entries
- MS/MS peak records per compound entry with precursor ion m/z, product ion m/z and intensity values
- retention time and polarity metadata per compound

## Outputs

- MRMPROBS-loadable transition table (tab/CSV) listing precursor m/z (Q1), product ion m/z (Q3), retention time, and polarity per compound
- exactly 5 precursor-product transition pairs per compound entry where 5 or more fragments are available

## How to apply

Load the curated in-house MS/MS library entries (449 positive-mode and 172 negative-mode plastic-related chemicals) from the verified msp output produced by mspcompiler's write_MS2_msp. For each entry, parse the precursor ion m/z and its MS/MS peak records, rank product ions by intensity, and retain the top 5 fragments to define precursor-product transition pairs (Q1→Q3). Write the transitions — precursor m/z, product ion m/z, retention time, and polarity — into a tab/CSV table loadable by MRMPROBS, which provides automated posterior probabilistic data assessment on the existing DIA data set without re-acquisition. Rationale: the pseudo-multiple reaction monitoring step is applied after untargeted identification to sensitively target and screen the 83 compounds identified in recycled HDPE milk bottles, sidestepping the need for triple-quadrupole standards run in true MRM mode.

## Related tools

- **mspcompiler** (produces the curated, verified msp MS/MS library whose entries are parsed for precursor and product-ion records) — https://github.com/QizhiSu/mspcompiler
- **MRMPROBS** (targeted-metabolomics software that loads the pseudo-MRM transition table and provides automated posterior probabilistic data assessment on DIA data)
- **MS-DIAL** (processed the DIA data (version 4.38) that the pseudo-MRM transitions are applied to)
- **R** (environment in which mspcompiler and the transition-pair extraction logic run)

## Examples

```
in_house <- read_multilibs("D:/MS_libraries/in_house"); write_MS2_msp(in_house, "D:/MS_libraries/in_house_ms2.msp")
```

## Evaluation signals

- The output file exists and is a loadable MRMPROBS transition table (parses as tab/CSV with precursor m/z, product ion m/z, RT, polarity columns).
- Every compound entry with 5 or more fragments contributes exactly 5 product ions; entries with fewer fragments contribute all available ions.
- The number of transition rows matches the number of library entries (449 positive-mode, 172 negative-mode) — no entries silently dropped.
- Each transition's precursor m/z equals the precursor ion m/z recorded for that compound in the source msp entry.
- Product ions selected are the highest-intensity fragments per compound (top-5 by intensity, not arbitrary).

## Limitations

- The transition pairs derive from library MS/MS spectra; fragments not represented in the library record will not be targeted.
- Retrospective pseudo-MRM assumes the DIA data were acquired with sufficient coverage of the precursor m/z and retention windows; re-acquisition would be needed if the original DIA parameters missed a compound.
- Only works for known unknowns already present in the compiled libraries — compounds absent from all source libraries cannot be targeted this way.

## Evidence

- [methods] MRMPROBS is an open-source software designed for targeted metabolomics, providing automated posterior probabilistic data assessment: "MRMPROBS is an open-source software designed for targeted metabolomics, providing automated posterior probabilistic data assessment"
- [abstract] A pseudo-multiple reaction monitoring method was also applied to sensitively target and screen the identified chemicals in the samples.: "A pseudo-multiple reaction monitoring method was also applied to sensitively target and screen the identified chemicals in the samples."
- [readme] read_multilibs function give you an easy way to read all of them at once: "The read_multilibs function give you an easy way to read all of them at once. In this case, what you need to input is the folder that contain all these msp files"
- [readme] write_MS2_msp(combine_ms2_pos, "D:MS_libraries/combine_ms2_pos.msp"): "# Then you can write them out separately
write_MS2_msp(combine_ms2_pos, "D:MS_libraries/combine_ms2_pos.msp")
write_MS2_msp(combine_ms2_neg, "D:MS_libraries/combine_ms2_neg.msp")"
- [readme] positive and negative modes are normally separated into 2 msp files: "MS2 libraries can be processed in a similar way, but positive and negative modes are normally separated into 2 msp files."
