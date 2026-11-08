---
name: dia-pseudomrm-targeting
description: Use when an untargeted LC-QTOF-MS/DIA run has produced identified chemicals
  (e.g. 83 compounds in rHDPE) and you need a sensitive targeted re-screening (pseudo-MRM)
  transition table for MRMPROBS instead of re-acquiring data.
when_to_use_negative:
- Identification is still ongoing — apply only after compounds are identified; the
  workflow states pseudo-MRM is applied 'after identification to sensitively target
  and screen the identified chemicals'.
- The dataset is a true triple-quadrupole MRM acquisition — this skill generates pseudo-MRM
  transitions for DIA data analysis in MRMPROBS, not a scheduled MRM method for instrument
  control.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mspcompiler
  role: Compiles and cleans the in-house MS/MS library into the msp file whose verified
    entries feed the transition export
  repo: https://github.com/QizhiSu/mspcompiler
- name: MRMPROBS
  role: Open-source targeted-metabolomics software that loads the pseudo-MRM transition
    table and performs automated posterior probabilistic data assessment on the DIA
    data
- name: MS-DIAL
  role: The DIA data processing environment whose identified features/library entries
    are the source of precursor and product ion records
- name: R
  role: Language in which mspcompiler and the library-export scripting are implemented
  repo: https://github.com/QizhiSu/mspcompiler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:70999ecb17505c4dd730f48f6d14ee5a88b015c41456299141913dc8a7ac3045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - MRMPROBS
  - MS-DIAL
  - R
  - mspcompiler
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# dia-pseudomrm-targeting

## Summary

Exports the curated in-house MS/MS library (mspcompiler-compiled msp entries for plastic-related chemicals) as an MRMPROBS pseudo-MRM library with the top 5 most intense product ions per compound, so identified migrants in DIA data from rHDPE milk bottles can be sensitively targeted and screened without re-acquisition.

## When to use

Use when an untargeted LC-QTOF-MS/DIA run has produced identified chemicals (e.g. 83 compounds in rHDPE) and you need a sensitive targeted re-screening (pseudo-MRM) transition table for MRMPROBS instead of re-acquiring data.

## When NOT to use

- Identification is still ongoing — apply only after compounds are identified; the workflow states pseudo-MRM is applied 'after identification to sensitively target and screen the identified chemicals'.
- The dataset is a true triple-quadrupole MRM acquisition — this skill generates pseudo-MRM transitions for DIA data analysis in MRMPROBS, not a scheduled MRM method for instrument control.

## Inputs

- mspcompiler-compiled in-house MS/MS library in msp format (449 positive-mode and 172 negative-mode plastic-related chemical entries)
- Verified MS/MS peak records per compound including precursor m/z, product ion m/z, intensity, and retention time

## Outputs

- MRMPROBS pseudo-MRM transition table (CSV/tab) with Q1 (precursor m/z), Q3 (product ion m/z), retention time, and polarity, top 5 product ions per compound
- Sensitively targeted pseudo-MRM screening of the identified chemicals on the existing DIA dataset

## How to apply

Load the curated in-house MS/MS library entries (449 positive-mode and 172 negative-mode plastic-related chemicals) from the verified msp output produced by mspcompiler. For each compound entry, parse the precursor ion m/z and its stored MS/MS peak records, rank product ions by intensity, and keep the top 5 fragments to form precursor-product transition pairs (Q1→Q3). Write the transitions — precursor m/z, product ion m/z, retention time, polarity — as a tab/CSV table loadable by MRMPROBS, the open-source targeted-metabolomics software that provides automated posterior probabilistic data assessment, so the DIA dataset can be interrogated as pseudo-MRM without new acquisition. Validate that the file is a loadable MRMPROBS transition table and contains exactly 5 product ions per compound wherever 5 or more fragments exist.

## Related tools

- **mspcompiler** (Compiles and cleans the in-house MS/MS library into the msp file whose verified entries feed the transition export) — https://github.com/QizhiSu/mspcompiler
- **MRMPROBS** (Open-source targeted-metabolomics software that loads the pseudo-MRM transition table and performs automated posterior probabilistic data assessment on the DIA data)
- **MS-DIAL** (The DIA data processing environment whose identified features/library entries are the source of precursor and product ion records)
- **R** (Language in which mspcompiler and the library-export scripting are implemented) — https://github.com/QizhiSu/mspcompiler

## Examples

```
write_MS2_msp(combine_ms2_pos, "D:/MS_libraries/combine_ms2_pos.msp")  # then export top-5 product-ion transitions to MRMPROBS format from the curated entries
```

## Evaluation signals

- Output file exists and is a loadable MRMPROBS transition table
- Every compound entry with ≥5 stored fragments has exactly 5 product ions (top-5-by-intensity selection)
- Each transition row carries precursor m/z, product ion m/z, retention time, and polarity consistent with the source msp entry
- Precursor m/z values match the identified compounds previously annotated via the in-house MS/MS library in MS-DIAL

## Limitations

- Transition selection depends on library spectra quality; compounds with fewer than 5 recorded product ions yield fewer than 5 transitions by design.
- Pseudo-MRM on DIA data cannot outperform genuine triple-quadrupole MRM sensitivity — the paper describes it as a way to 'sensitively target and screen' already-identified chemicals.
- The strategy relies on a curated in-house library (more than 500 plastic-related chemicals); compounds absent from the library cannot be targeted this way.

## Evidence

- [methods] Pseudo-MRM applied after identification to sensitively target chemicals: "MRMPROBS is an open-source software designed for targeted metabolomics, providing automated posterior probabilistic data assessment"
- [abstract] Pseudo-MRM screening of identified chemicals in samples: "A pseudo-multiple reaction monitoring method was also applied to sensitively target and screen the identified chemicals in the samples."
- [abstract] In-house MS/MS library open-sourced: "we built and open-sourced an in-house MS/MS library containing more than 500 plastic-related chemicals"
- [intro] mspcompiler output feeds MS-DIAL identification: "organize them into a neat and up-to-date msp file that can be used in MS-DIAL"
- [readme] README confirms msp files are the compiled output consumed downstream: "organize them into a neat and up-to-date msp file that can be used in MS-DIAL."
