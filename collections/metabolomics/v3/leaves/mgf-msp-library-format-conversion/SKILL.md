---
name: mgf-msp-library-format-conversion
description: Use when building an MS-DIAL identification library from multiple public
  tandem-MS sources in mixed formats (msp and mgf) that need polarity separation and
  structure-field harmonization.
when_to_use_negative:
- Input is already a combined, MS-DIAL-parseable msp file with structure fields intact
- Working with EI (GC-MS) libraries — use the EI-specific pipeline (read_lib type='EI',
  extract_structure, assign_ri) instead
- Spectra need per-compound retention-time alignment rather than library compilation
  — mspcompiler handles library assembly, not chromatographic RT alignment
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mspcompiler
  role: R package providing read_lib, reorganize_mona, complete_mgf, separate_polarity,
    write_MS2_msp, and read_multilibs for the whole conversion
  repo: https://github.com/QizhiSu/mspcompiler
- name: MS-DIAL
  role: Downstream consumer of the compiled msp library for feature identification
- name: Lib2NIST
  role: Pre-step to export NIST MSMS (and SWGDRUG) into msp + MOL format readable
    by read_lib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:d5f75aff46748a3068f395c7d0e66df14ea82b9322f8152e98dbceff1b519ff8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - Lib2NIST
  - MS-DIAL
  - complete_mgf
  - mspcompiler
  - read_lib
  - reorganize_mona
  - write_MS2_msp
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# mgf-msp-library-format-conversion

## Summary

Compile tandem-MS libraries from heterogeneous public sources (NIST, MoNA, GNPS, RIKEN) into a unified MS-DIAL-ready msp file using the mspcompiler R package, harmonizing structure fields (SMILES, InChIKey, molecular formula) and separating polarities.

## When to use

Use when building an MS-DIAL identification library from multiple public tandem-MS sources in mixed formats (msp and mgf) that need polarity separation and structure-field harmonization.

## When NOT to use

- Input is already a combined, MS-DIAL-parseable msp file with structure fields intact
- Working with EI (GC-MS) libraries — use the EI-specific pipeline (read_lib type='EI', extract_structure, assign_ri) instead
- Spectra need per-compound retention-time alignment rather than library compilation — mspcompiler handles library assembly, not chromatographic RT alignment

## Inputs

- MoNA LC-MS/MS Positive Mode and Negative Mode msp exports
- GNPS 'All GNPS Library Spectra' mgf file
- NIST MSMS msp + MOL folder exported via Lib2NIST
- RIKEN MSMS-Public-Pos-VS15 / MSMS-Public-Neg-VS15 msp files
- In-house batch msp files in a single folder

## Outputs

- combine_ms2_pos.msp
- combine_ms2_neg.msp
- MS-DIAL-parseable tandem-MS library entries with SMILES, InChIKey, and Molecular Formula fields populated

## How to apply

Read each source with read_lib, setting type='MS2' and format='mgf' only for GNPS 'All GNPS Library Spectra'. Apply reorganize_mona to MoNA exports to move SMILES out of the Comment field; run complete_mgf on GNPS to compute the missing Molecular Formula field from SMILES; use separate_polarity on NIST MSMS and GNPS files that mix positive and negative modes; then concatenate per polarity and write out with write_MS2_msp as combine_ms2_pos.msp and combine_ms2_neg.msp. For the in-house msp files in a folder, read_multilibs reads them all at once. The output msp files must be parseable by MS-DIAL (version 4.38 was used downstream) with each record carrying polarity-separated, SMILES- and MF-annotated spectra.

## Related tools

- **mspcompiler** (R package providing read_lib, reorganize_mona, complete_mgf, separate_polarity, write_MS2_msp, and read_multilibs for the whole conversion) — https://github.com/QizhiSu/mspcompiler
- **MS-DIAL** (Downstream consumer of the compiled msp library for feature identification)
- **Lib2NIST** (Pre-step to export NIST MSMS (and SWGDRUG) into msp + MOL format readable by read_lib)

## Examples

```
gnps <- read_lib('D:/MS_libraries/ALL_GNPS.mgf', format='mgf'); gnps <- complete_mgf(gnps); gnps_pos <- separate_polarity(gnps, polarity='pos'); write_MS2_msp(gnps_pos, 'D:/MS_libraries/combine_ms2_pos.msp')
```

## Evaluation signals

- Both combine_ms2_pos.msp and combine_ms2_neg.msp exist and are non-trivially sized
- Each output msp file parses in MS-DIAL without structural errors
- Every spectrum entry has a polarity-consistent annotation matching its source file's mode
- GNPS-derived records have a computed Molecular Formula field (no empty MF)
- MoNA-derived records carry SMILES in the SMILES field, not in Comment

## Limitations

- GNPS records lacking SMILES cannot get a Molecular Formula — complete_mgf computes MF 'from the SMILES (if it exists)' only
- ChemineR/ChemmineOB installation can fail and needs BiocManager fallback
- Very large NIST msp + MOL exports take hours and benefit from parallel computing
- mspcompiler is a compiler — it does not filter or score spectra against experimental data

## Evidence

- [readme] The goal of mspcompiler is to offer ways to compile either EI or tandem mass spectral libraries from various sources, such as NIST (if you have it installed), MoNA, and GPNS, and organize them into a neat and up-to-date msp file that can be used in MS-DIAL.: [evidence span withheld — anchor status: not_found]
- [readme] Unlike others, the GNPS library is organized in mgf format, so it has to be treated differently. Hence, we have to set format = 'mgf' in the read_lib function. Besides, this library does not have the Molecular Formula (MF) field, so we can calculated the MF from the SMILES (if it exists) by the complete_mgf function.: "the GNPS library is organized in mgf format, so it has to be treated differently. Hence, we have to set *format = 'mgf'* in the *read_lib* function. Besides, this library does not have the *Molecular"
- [readme] the SMILES has to be extracted from the Comment and put into the SMILES field by the reorganize_mona function.: "the SMILES has to be extracted from the *Comment* and put into the *SMILES* field by the *reorganize\_mona* function."
- [readme] The exported msp file has both positive and negative modes mixed in a singled file, so we have to separated them by the separate_polarity function.: "The exported msp file has both positive and negative modes mixed in a singled file, so we have to separated them by the separate\_polarity function."
- [abstract] developed mspcompiler, an R package, for the compilation of various libraries: [evidence span withheld — anchor status: not_found]
