---
name: library-composition-verification
description: Use when you have downloaded a public MS/MS library (e.g. from github.com/QizhiSu/MS-libraries)
  and must confirm its entry counts per polarity match the reported composition before
  feeding it to MS-DIAL identification.
when_to_use_negative:
- Input is already a verified or vendor-issued library whose counts are guaranteed
  — re-counting adds no information.
- You need to re-annotate, merge, or add SMILES/RI to library entries — that is mspcompiler's
  read_lib/assign_smiles/assign_ri pipeline, not composition verification.
- The library is in mgf format (e.g. GNPS ALL_GNPS.mgf) without published count expectations
  — use read_lib(format="mgf") plus complete_mgf rather than this count-vs-paper check.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mspcompiler
  role: R package whose read_lib (format="msp", type="MS2") and read_multilibs functions
    parse the MS/MS library files so entries can be counted per polarity; write_EI_msp/write_MS2_msp
    or plain readr exports the verification table.
  repo: https://github.com/QizhiSu/mspcompiler
- name: R
  role: runtime environment in which mspcompiler is installed (via devtools::install_github)
    and where the count-comparison and readr table-writing script runs.
  repo: https://github.com/QizhiSu/mspcompiler
- name: readr
  role: used to write the verification report table (ionization mode, observed count,
    expected count, match status).
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:700647c4c8c711e1eabfcfb60552020227f47cdd511891332e49cc2f0e7dcb0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - R
  - mspcompiler
  - read_lib
  - readr
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# library-composition-verification

## Summary

Verify the composition of a downloaded or compiled MS/MS spectral library by reading its entries (e.g. with mspcompiler's read_lib/read_multilibs) and counting records per ionization mode against the published expected counts. This confirms that an openly deposited in-house library (here 449 positive-mode and 172 negative-mode plastic-related entries, >500 total) matches what the paper reports before it is used for identification.

## When to use

Use when you have downloaded a public MS/MS library (e.g. from github.com/QizhiSu/MS-libraries) and must confirm its entry counts per polarity match the reported composition before feeding it to MS-DIAL identification.

## When NOT to use

- Input is already a verified or vendor-issued library whose counts are guaranteed — re-counting adds no information.
- You need to re-annotate, merge, or add SMILES/RI to library entries — that is mspcompiler's read_lib/assign_smiles/assign_ri pipeline, not composition verification.
- The library is in mgf format (e.g. GNPS ALL_GNPS.mgf) without published count expectations — use read_lib(format="mgf") plus complete_mgf rather than this count-vs-paper check.

## Inputs

- in-house MS/MS library files (msp format) in positive and negative ionization modes, downloaded from the public repository
- expected entry counts from the paper's methods section (e.g. 449 positive-mode, 172 negative-mode)
- optional: folder containing multiple per-batch msp files for read_multilibs

## Outputs

- verified MS/MS library files (one per polarity)
- counts table with columns for ionization mode, observed entry count, expected entry count, and match status (written with readr)

## How to apply

Download the MS/MS library files from the public repository (here https://github.com/QizhiSu/MS-libraries). Read each polarity file separately with mspcompiler's read_lib, setting format = "msp" and type = "MS2" (or read_multilibs when the in-house library is split across many per-batch msp files in one folder). Count the number of spectral entries in the positive-mode file and compare it to the expected value (449 for the plastic-related in-house library), then count the negative-mode entries and compare to 172; total >500 entries is consistent with the paper's claim of 'more than 500 plastic-related chemicals'. Record the observed vs expected counts and a match status in a small table written with readr, and save the verified library files for downstream MS-DIAL identification. Validation requires exact equality of counts with the reported values; any mismatch means the wrong file version was downloaded or the file was truncated during read_lib.

## Related tools

- **mspcompiler** (R package whose read_lib (format="msp", type="MS2") and read_multilibs functions parse the MS/MS library files so entries can be counted per polarity; write_EI_msp/write_MS2_msp or plain readr exports the verification table.) — https://github.com/QizhiSu/mspcompiler
- **R** (runtime environment in which mspcompiler is installed (via devtools::install_github) and where the count-comparison and readr table-writing script runs.) — https://github.com/QizhiSu/mspcompiler
- **readr** (used to write the verification report table (ionization mode, observed count, expected count, match status).)

## Examples

```
nist_ms2 <- read_lib("D:/MS_libraries/NIST_msms.MSP", type = "MS2"); nist_ms2_pos <- separate_polarity(nist_ms2, polarity = "pos"); length(nist_ms2_pos)
```

## Evaluation signals

- Observed positive-mode entry count equals the paper's reported value exactly (449).
- Observed negative-mode entry count equals the paper's reported value exactly (172).
- Combined total exceeds 500 entries, consistent with the abstract's claim of 'more than 500 plastic-related chemicals'.
- The verification table contains a match-status column recording 'match' for both polarities.
- Verified library files are saved and loadable by MS-DIAL without further parsing errors.

## Limitations

- The strategy only works for known unknowns — if the library lacks an entry for a compound of interest, composition verification will not surface that gap.
- Counting requires correctly separated polarity files; mixed-polarity exports (e.g. from NIST MS2 via Lib2NIST) must first be split with separate_polarity before counts are meaningful.
- read_lib of very large msp files is time-consuming; mspcompiler's README recommends parallel computing (plan(multisession)) and setting it back with plan(sequential) afterwards.

## Evidence

- [methods] In-house MS/MS library composition counts: "the SMILES has to be extracted from the *Comment* and put into the *SMILES* field by the *reorganize_mona* function."
- [abstract] More than 500 plastic-related chemicals claimed: "we built and open-sourced an in-house MS/MS library containing more than 500 plastic-related chemicals"
- [intro] Library must be compileable into msp for MS-DIAL: "compile either EI or tandem mass spectral libraries from various sources"
- [readme] Mixed-polarity export must be split before counting: "The exported msp file has both positive and negative modes mixed in a singled file, so we have to separated them by the separate_polarity function."
- [readme] read_multilibs handles many per-batch msp files: "The read_multilibs function give you an easy way to read all of them at once."
