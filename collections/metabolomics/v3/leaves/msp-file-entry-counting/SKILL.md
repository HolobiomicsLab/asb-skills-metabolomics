---
name: msp-file-entry-counting
description: Use when an msp or mgf spectral library file must be loaded into R and
  its number of spectral entries counted, especially to verify that a deposited in-house
  library matches a paper-reported composition.
when_to_use_negative:
- The input is already a compiled or cleaned feature table (e.g. MS-DIAL output cleaned
  by MS-CleanR) rather than a raw spectral library file — counting should have happened
  upstream at the library-compilation stage.
- You need to add SMILES, RI, or MF fields to the library rather than just counting
  entries — use the assign_smiles / extract_ri / complete_mgf functions instead.
- The library is a single mixed-polarity file and you intend to keep it combined —
  read_lib alone will not split modes; only apply this skill when per-mode counts
  are the goal.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mspcompiler
  role: R package whose read_lib, read_multilibs, and separate_polarity functions
    parse the msp/mgf library files so entries can be counted per ionization mode
  repo: https://github.com/QizhiSu/mspcompiler
- name: R
  role: Runtime language in which mspcompiler runs and the counts are computed and
    compared to expected values
- name: readr
  role: Used to write the verification report table of observed vs expected entry
    counts and match status
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:615393ed99db363e3188567ae24d6cdc82630ad8bf43fb3bd657cd77f9d307ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - R
  - mspcompiler
  - read_lib
  - readr
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# msp-file-entry-counting

## Summary

Reads msp-format MS/MS library files with mspcompiler's read_lib and counts spectral entries per ionization mode to verify a library's composition against reported counts (e.g. 449 positive-mode and 172 negative-mode plastic-related chemical entries).

## When to use

Use when an msp or mgf spectral library file must be loaded into R and its number of spectral entries counted, especially to verify that a deposited in-house library matches a paper-reported composition.

## When NOT to use

- The input is already a compiled or cleaned feature table (e.g. MS-DIAL output cleaned by MS-CleanR) rather than a raw spectral library file — counting should have happened upstream at the library-compilation stage.
- You need to add SMILES, RI, or MF fields to the library rather than just counting entries — use the assign_smiles / extract_ri / complete_mgf functions instead.
- The library is a single mixed-polarity file and you intend to keep it combined — read_lib alone will not split modes; only apply this skill when per-mode counts are the goal.

## Inputs

- msp-format MS/MS library file (e.g. in-house library of plastic-related chemicals, or combine_ms2_pos.msp / combine_ms2_neg.msp produced by mspcompiler)
- mgf-format library file (e.g. ALL_GNPS.mgf) when counting GNPS-sourced entries
- expected entry counts per ionization mode taken from the paper's methods section (e.g. 449 positive, 172 negative)

## Outputs

- Per-mode spectral entry counts (integer) for positive and negative ionization
- Verification table (via readr) with columns for ionization mode, observed count, expected count, and match status
- Verified library objects in R, ready for downstream MS-DIAL annotation

## How to apply

Install mspcompiler from GitHub and read each library file with read_lib, specifying format = 'msp' and type = 'MS2' for tandem-MS libraries (or type = 'EI' for GC-EI libraries; format = 'mgf' for GNPS-style mgf files). Because MS/MS libraries are typically split by polarity, read the positive-mode and negative-mode files separately — for mixed-polarity sources such as NIST MS/MS or GNPS, first apply separate_polarity to split the combined file into pos and neg subsets. Count the number of entries in each returned object (each entry is one spectrum record) and compare against expected values, e.g. 449 for the positive-mode and 172 for the negative-mode plastic-related in-house library. When multiple per-batch msp files make up an in-house library, use read_multilibs to read all of them at once from the folder path. Write the observed vs expected counts to a verification table with readr, recording a match status per ionization mode, and save the verified library files for downstream MS-DIAL identification tasks.

## Related tools

- **mspcompiler** (R package whose read_lib, read_multilibs, and separate_polarity functions parse the msp/mgf library files so entries can be counted per ionization mode) — https://github.com/QizhiSu/mspcompiler
- **R** (Runtime language in which mspcompiler runs and the counts are computed and compared to expected values)
- **readr** (Used to write the verification report table of observed vs expected entry counts and match status)

## Examples

```
library(mspcompiler); lib <- read_lib('in_house_msms.msp', type='MS2'); length(lib)
```

## Evaluation signals

- Observed entry count equals the paper-reported value exactly (449 for positive mode, 172 for negative mode of the in-house plastic-related library).
- read_lib is called with format = 'msp' and type = 'MS2' (or format = 'mgf' for GNPS) — mismatched type/format flags a parsing error.
- For mixed-polarity source files, separate_polarity must have been applied before per-mode counting; counting a mixed file directly would double-count one mode.
- The verification table records a match status per mode and both rows show a match; any mismatch indicates either a wrong file version or a parsing failure.
- Total counted entries (positive + negative) should exceed 500, consistent with the paper's 'more than 500 plastic-related chemicals' statement.

## Limitations

- The workflow's identification strategy only works for known unknowns; unknown unknowns may not be present in any existing structure databases, so a matched entry count does not guarantee every sample feature can be annotated.
- Reading very large msp files (e.g. NIST exports of hundreds of megabytes) is time-consuming; mspcompiler's README recommends parallel computing with future/multisession and resetting plan(sequential) afterwards.
- Library files must be obtained from the correct public source (e.g. https://github.com/QizhiSu/MS-libraries) — the counts depend on the deposited version of the in-house library.

## Evidence

- [methods] read_lib: "The read_multilibs function give 
you an easy way to read all of them at once."
