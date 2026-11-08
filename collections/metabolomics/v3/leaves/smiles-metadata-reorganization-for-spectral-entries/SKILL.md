---
name: smiles-metadata-reorganization-for-spectral-entries
description: Use when parsing MoNA msp exports (or GNPS 'All GNPS Library Spectra'
  mgf) whose SMILES/MF metadata is misplaced or absent, and the records must be structurally
  annotated before writing an MS-DIAL-ready msp.
when_to_use_negative:
- The library already has SMILES/InChIKey in canonical fields, such as the RIKEN MS-DIAL
  exports (MSMS-Public-Pos-VS15.msp) — reorganization is unnecessary and read_lib
  alone suffices
- The source is a NIST or SWGDRUG msp paired with .MOL files, where structures must
  instead be recovered via combine_mol2sdf → extract_structure → assign_smiles rather
  than from a Comment field
- The goal is plain library parsing without structural annotation, where metadata
  reorganization adds no value
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mspcompiler
  role: R package supplying read_lib, reorganize_mona, complete_mgf, separate_polarity,
    and write_MS2_msp used to normalize and compile the spectral library metadata
  repo: https://github.com/QizhiSu/mspcompiler
- name: MS-DIAL
  role: Downstream consumer of the reorganized msp file, using the normalized SMILES/MF
    fields for identification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:fae32be50b030de42cc887e3255f70bc91c6d4d282137aebfa54a86d2e08dd50
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
  - write_MS2_msp
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# smiles-metadata-reorganization-for-spectral-entries

## Summary

Reorganizes MoNA-exported mass spectral library records so that SMILES strings stored in the Comment field are moved into a dedicated SMILES field (and, for GNPS mgf entries, computes the missing Molecular Formula from SMILES) before compiling libraries with mspcompiler for MS-DIAL.

## When to use

Use when parsing MoNA msp exports (or GNPS 'All GNPS Library Spectra' mgf) whose SMILES/MF metadata is misplaced or absent, and the records must be structurally annotated before writing an MS-DIAL-ready msp.

## When NOT to use

- The library already has SMILES/InChIKey in canonical fields, such as the RIKEN MS-DIAL exports (MSMS-Public-Pos-VS15.msp) — reorganization is unnecessary and read_lib alone suffices
- The source is a NIST or SWGDRUG msp paired with .MOL files, where structures must instead be recovered via combine_mol2sdf → extract_structure → assign_smiles rather than from a Comment field
- The goal is plain library parsing without structural annotation, where metadata reorganization adds no value

## Inputs

- MoNA LC-MS/MS Positive/Negative Mode msp export files (e.g. MoNA-export-LC-MS-MS_Positive_Mode.msp)
- MoNA GC-MS Spectra msp export (e.g. MoNA-export-GC-MS_Spectra.msp)
- GNPS 'All GNPS Library Spectra' mgf file (e.g. ALL_GNPS.mgf)
- NIST MS2 msp export with mixed polarity (optional, for the separate_polarity step)

## Outputs

- In-memory spectral library records with SMILES populated in the SMILES field (no longer in Comment)
- GNPS mgf records with computed Molecular Formula field
- Polarity-separated library objects ready to be combined and written via write_MS2_msp (e.g. combine_ms2_pos.msp, combine_ms2_neg.msp)

## How to apply

Read the raw library file with mspcompiler's read_lib — type = "EI" for MoNA GC-MS exports, type = "MS2" for MoNA LC-MS/MS exports, or format = "mgf" for the GNPS 'All GNPS Library Spectra' file. For MoNA records, apply the reorganize_mona function, which extracts the SMILES string from the Comment field and writes it into the SMILES field so downstream structure-aware steps (e.g. formula computation, polarity separation, MS-DIAL matching) can use it directly. For GNPS mgf entries lacking a Molecular Formula field, apply complete_mgf to calculate MF from existing SMILES; then split mixed-mode records with separate_polarity (polarity = "pos" or "neg") because MS2 libraries are written as separate positive and negative msp files. Parallelize with future/plan(multisession(...)) and reset to plan(sequential) before writing out with write_MS2_msp or write_EI_msp. The rationale is that MS-DIAL library matching relies on per-spectrum metadata fields — SMILES, InChIKey, and MF — being present in their canonical positions, so annotations cannot propagate unless the raw exports are normalized first.

## Related tools

- **mspcompiler** (R package supplying read_lib, reorganize_mona, complete_mgf, separate_polarity, and write_MS2_msp used to normalize and compile the spectral library metadata) — https://github.com/QizhiSu/mspcompiler
- **MS-DIAL** (Downstream consumer of the reorganized msp file, using the normalized SMILES/MF fields for identification)

## Examples

```
library(mspcompiler); mona_ms2_pos <- read_lib("D:/MS_libraries/MoNA-export-LC-MS-MS_Positive_Mode.msp"); mona_ms2_pos <- reorganize_mona(mona_ms2_pos)
```

## Evaluation signals

- After reorganize_mona, every parsed record has a non-empty SMILES field and the Comment field no longer carries the SMILES string
- After complete_mgf, each GNPS entry that had a SMILES string has a filled Molecular Formula field consistent with that SMILES
- separate_polarity produces two mutually exclusive record sets (pos and neg) whose union equals the input library, with no mode mixing in either output
- The final write_MS2_msp output (e.g. combine_ms2_pos.msp / combine_ms2_neg.msp) is a parseable MS-DIAL msp text file retaining polarity-separated, SMILES- and formula-annotated records
- Record counts before and after reorganization match — no spectra are dropped or duplicated by the metadata move

## Limitations

- For Linux/Mac NIST processing, structure matching must use match = "inchikey" rather than "name" — but SWGDRUG lacks InChIKey, so name matching must be used there regardless of platform
- complete_mgf can only compute MF when a SMILES string exists in the entry; GNPS records without SMILES remain formula-less
- Lib2NIST-exported msp files are large (hundreds of MB with hundreds of thousands of .MOL files), making the surrounding pipeline time-consuming and requiring parallel computing
- The skill addresses only metadata placement/polarity splitting; it does not add structures to libraries like NIST/SWGDRUG that carry structures in separate MOL/SDF files

## Evidence

- [readme] MoNA Comment-field SMILES extraction: "This file has SMILES information though, it is in the *Comment* field. Therefore, the SMILES has to be extracted from the *Comment* and put into the *SMILES* field by the *reorganize_mon a* function."
- [readme] GNPS mgf lacks MF; compute from SMILES: "this library does not have the *Molecular Formula* (MF) field, so we can calculated the MF from the SMILES (if it exists) by the *complete_mgf* function."
- [readme] Mixed polarity requires splitting: "both positive and negative modes are in a single file as well. Therefore, we need to separated the polarity by the *separate_polarity* function as well."
- [readme] Purpose of mspcompiler: "organize them into a neat and up-to-date msp file that can be used in MS-DIAL."
- [methods] Article description of reorganize_mona: "the SMILES has to be extracted from the *Comment* and put into the *SMILES* field by the *reorganize_mona* function."
- [readme] RIKEN exports already normalized: "This file already have SMILES and InChIKey well-organized. Hence, no further treatment is needed."
