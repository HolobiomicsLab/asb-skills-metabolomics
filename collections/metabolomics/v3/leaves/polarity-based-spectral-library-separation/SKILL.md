---
name: polarity-based-spectral-library-separation
description: Use when a tandem-MS library (e.g. NIST_msms.MSP or ALL_GNPS.mgf) mixes
  positive and negative ionization entries in a single file that must be split before
  MS-DIAL identification.
when_to_use_negative:
- The library is already polarity-resolved, e.g. RIKEN MSMS-Public-Pos-VS15.msp and
  MSMS-Public-Neg-VS15.msp are downloaded as separate positive and negative files,
  or MoNA LC-MS/MS Positive Mode and Negative Mode exports
tools:
- name: mspcompiler
  role: R package providing the separate_polarity function that splits a parsed MS2
    library into positive and negative entries
  repo: https://github.com/QizhiSu/mspcompiler
- name: read_lib
  role: mspcompiler function that parses NIST msp or GNPS mgf files into a library
    object ready for polarity separation
  repo: https://github.com/QizhiSu/mspcompiler
- name: complete_mgf
  role: mspcompiler function that computes the Molecular Formula field from SMILES
    for GNPS mgf entries before polarity separation
  repo: https://github.com/QizhiSu/mspcompiler
- name: write_MS2_msp
  role: mspcompiler function that writes each polarity-separated combined library
    to an MS-DIAL-ready msp file
  repo: https://github.com/QizhiSu/mspcompiler
- name: MS-DIAL
  role: Downstream identification engine (version 4.38 in this study) that consumes
    the separate polarity msp files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:abcd771995d4b0b1270a89972a95b583defbef5cfaaee5f8e70a6a90c903fc2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - MS-DIAL
  - complete_mgf
  - mspcompiler
  - read_lib
  - reorganize_mona
  - separate_polarity
  - write_MS2_msp
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# polarity-based-spectral-library-separation

## Summary

Separates a mixed-ion-mode tandem MS spectral library into positive- and negative-mode msp files using mspcompiler's separate_polarity, so each polarity can be combined and searched independently in MS-DIAL.

## When to use

Use when a tandem-MS library (e.g. NIST_msms.MSP or ALL_GNPS.mgf) mixes positive and negative ionization entries in a single file that must be split before MS-DIAL identification.

## When NOT to use

- The library is already polarity-resolved, e.g. RIKEN MSMS-Public-Pos-VS15.msp and MSMS-Public-Neg-VS15.msp are downloaded as separate positive and negative files, or MoNA LC-MS/MS Positive Mode and Negative Mode exports

## Inputs

- MIXED-POLARITY tandem-MS library file (e.g. NIST_msms.MSP from Lib2NIST export, or ALL_GNPS.mgf from GNPS 'All GNPS Library Spectra') parsed via read_lib

## Outputs

- Positive-mode msp library (combine_ms2_pos.msp) writable by write_MS2_msp for MS-DIAL
- Negative-mode msp library (combine_ms2_neg.msp) writable by write_MS2_msp for MS-DIAL

## How to apply

Read the mixed library with read_lib (type MS2; format mgf for GNPS) into an R session running mspcompiler under parallel computing (plan(multisession(workers = detectCores() - 1))). For GNPS mgf input, first compute the missing Molecular Formula field from SMILES with complete_mgf, since mgf entries lack an MF field. Then call separate_polarity twice on the parsed library: once with polarity = 'pos' and once with polarity = 'neg', producing two polarity-disjoint lists of spectral entries. Concatenate each polarity's entries with the corresponding MoNA or RIKEN positive/negative libraries using c(), and write out combine_ms2_pos.msp and combine_ms2_neg.msp via write_MS2_msp so MS-DIAL matches precursor/product ions against polarity-consistent references.

## Related tools

- **mspcompiler** (R package providing the separate_polarity function that splits a parsed MS2 library into positive and negative entries) — https://github.com/QizhiSu/mspcompiler
- **read_lib** (mspcompiler function that parses NIST msp or GNPS mgf files into a library object ready for polarity separation) — https://github.com/QizhiSu/mspcompiler
- **complete_mgf** (mspcompiler function that computes the Molecular Formula field from SMILES for GNPS mgf entries before polarity separation) — https://github.com/QizhiSu/mspcompiler
- **write_MS2_msp** (mspcompiler function that writes each polarity-separated combined library to an MS-DIAL-ready msp file) — https://github.com/QizhiSu/mspcompiler
- **MS-DIAL** (Downstream identification engine (version 4.38 in this study) that consumes the separate polarity msp files)

## Examples

```
gnps <- read_lib("D:/MS_libraries/ALL_GNPS.mgf", format = "mgf"); gnps <- complete_mgf(gnps); gnps_pos <- separate_polarity(gnps, polarity = "pos"); gnps_neg <- separate_polarity(gnps, polarity = "neg")
```

## Evaluation signals

- After separate_polarity(pos) and separate_polarity(neg), the two resulting entry lists must be disjoint: no spectrum record appears in both combine_ms2_pos and combine_ms2_neg.
- Each output msp must be parseable MS-DIAL msp text and every GNPS-derived entry should retain a computed Molecular Formula field from complete_mgf.
- Entry counts in the positive and negative outputs should sum to the number of entries in the input mixed library (no spectra lost).
- MS-DIAL (v4.38) should load both combine_ms2_pos.msp and combine_ms2_neg.msp without polarity-mismatched annotations.

## Limitations

- The NIST MS2 msp exported by Lib2NIST mixes both ion modes in a single file, so separation is mandatory before combination; polarity must not be inferred silently.
- GNPS mgf entries lack the Molecular Formula field, so polarity separation alone does not produce an MF-complete library — complete_mgf must run first.
- The workflow described applies to LC-MS/MS (tandem) libraries; EI libraries follow a different, RI-assignment-focused path.

## Evidence

- [readme] NIST MS2 polarity split is required before combination: "The exported msp file has both positive and negative modes mixed in a singled file, so we have to separated them by the separate_polarity function."
- [readme] GNPS mgf lacks MF; computed from SMILES before splitting: "this library does not have the *Molecular Formula* (MF) field, so we can calculated the MF from the SMILES (if it exists) by the *complete_mgf* function."
- [readme] Combined polarity libraries are written as separate msp files: "write_MS2_msp(combine_ms2_pos, "D:MS_libraries/combine_ms2_pos.msp")"
- [readme] GNPS library is mgf-format and needs format = mgf in read_lib: "Unlike others, the GNPS library is organized in mgf format, so it has to be treated differently. Hence, we have to set *format = "mgf"* in the *read_lib* function."
- [intro] mspcompiler compiles libraries into an MS-DIAL-ready msp file: [evidence span withheld — anchor status: not_found]
