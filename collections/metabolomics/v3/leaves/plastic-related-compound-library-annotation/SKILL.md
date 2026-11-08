---
name: plastic-related-compound-library-annotation
description: Use when untargeted LC-HRMS (DIA) feature lists from polymer/food-contact
  samples must be annotated against a curated MS/MS library of plastic-related chemicals,
  and raw library exports (NIST, MoNA, GNPS, RIKEN, SWGDRUG) lack SMILES/RI fields
  needed by MS-DIAL.
when_to_use_negative:
- Identifying unknown unknowns that may not be present in any existing structure database
  — the in silico fragmentation/library strategy only works for known unknowns.
- Quantification or pseudo-MRM targeting of already-identified compounds — use MRMPROBS
  after annotation, not library compilation.
- Single-source libraries already containing well-organized SMILES and RI (e.g. RIKEN
  'All records with Kovats RI…EI-MS…') need no reorganization beyond read_lib.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: mspcompiler
  role: R package that reads, structures-annotates (SMILES/RI/MF), polarity-splits,
    and writes organized msp files from NIST, MoNA, GNPS, RIKEN, SWGDRUG, and in-house
    libraries for MS-DIAL identification
  repo: https://github.com/QizhiSu/mspcompiler
- name: MS-DIAL 4.38
  role: Performs untargeted DIA feature detection and MS/MS library matching against
    the compiled in-house msp library on UPLC-QTOF-MS data
- name: Lib2NIST
  role: Converts installed NIST (and SWGDRUG) libraries into Text File(.MSP) + MOLfiles
    format suitable for mspcompiler parsing
  repo: https://chemdata.nist.gov/dokuwiki/doku.php?id=chemdata:nist17
- name: MS-CleanR
  role: Cleans MS-DIAL-exported feature tables (blank/QC removal, RSD > 30 discarded)
    prior to annotation and structural elucidation
- name: MS-FINDER
  role: Fallback in silico fragmentation for unknowns not annotated by the MS/MS library,
    using volDB, cppDB, and genDB structure databases
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:087c348637f40d03626192f3f982c0df5018de5edff80930cb444952f1038df0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - Lib2NIST
  - MRMPROBS
  - MS-CleanR
  - MS-DIAL
  - MS-DIAL 4.38
  - MS-FINDER
  - R
  - mspcompiler
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# Plastic-related compound library annotation

## Summary

Compiles and enriches an in-house MS/MS library of >500 plastic-related chemicals with SMILES, retention indices, and cleaned spectra (mspcompiler) so that MS-DIAL can annotate non-volatile migrants from recycled HDPE; in the source study 66 of 83 identified compounds were annotated this way.

## When to use

Use when untargeted LC-HRMS (DIA) feature lists from polymer/food-contact samples must be annotated against a curated MS/MS library of plastic-related chemicals, and raw library exports (NIST, MoNA, GNPS, RIKEN, SWGDRUG) lack SMILES/RI fields needed by MS-DIAL.

## When NOT to use

- Identifying unknown unknowns that may not be present in any existing structure database — the in silico fragmentation/library strategy only works for known unknowns.
- Quantification or pseudo-MRM targeting of already-identified compounds — use MRMPROBS after annotation, not library compilation.
- Single-source libraries already containing well-organized SMILES and RI (e.g. RIKEN 'All records with Kovats RI…EI-MS…') need no reorganization beyond read_lib.

## Inputs

- Raw MS/MS or EI library exports: NIST .MSP + .MOL folder, NIST ri.dat and USER.DBU, MoNA GC-MS and LC-MS/MS msp, RIKEN MS-DIAL EI and MS2 msp, GNPS ALL_GNPS.mgf, SWGDRUG NIST/Agilent formats, in-house per-batch standard msp files
- QC/blank-filtered feature tables from MS-DIAL (cleaned by MS-CleanR with RSD cutoff 30)
- mzML/DIA acquisition files from UPLC-QTOF-MS (Waters Acquity UPLC, Atlantis Premier BEH C18 AX column)

## Outputs

- Annotated feature tables (compound identifications/annotations per sample) from MS-DIAL against the compiled msp library
- Cleaned, SMILES- and RI-enriched combined EI and MS2 msp files (e.g. combine_ms2_pos.msp, combine_ms2_neg.msp) loadable in MS-DIAL
- Verified in-house MS/MS library containing more than 500 plastic-related chemicals (449 positive-mode and 172 negative-mode entries)

## How to apply

Read raw library exports into R with mspcompiler's read_lib/read_multilibs (EI vs MS2 type, mgf format for GNPS), convert NIST/Agilent sources via Lib2NIST to .MSP plus .MOL folders, then combine_mol2sdf and extract_structure to retrieve structures from the MOL/sdf files, and assign_smiles (match = "name", or "inchikey" on Linux/macOS when InChIKey exists) to fill the SMILES field. Reorganize MoNA SMILES from the Comment field with reorganize_mona, compute missing GNPS molecular formulas with complete_mgf, and split mixed-polarity NIST MS2/GNPS libraries with separate_polarity into separate pos/neg msp files. For EI libraries, add experimental retention indices with extract_ri on NIST's ri.dat and USER.DBU files and assign_ri with polarity set to "semi-polar", which keeps only capillary-column records, drops Lee RI, takes the median when multiple records exist, and discards entries whose SD exceeds 30. Concatenate libraries with c() and write out with write_EI_msp/write_MS2_msp; the resulting organized msp files are loaded by MS-DIAL 4.38, where features surviving MS-CleanR cleaning (blank/QC removal, RSD cutoff 30) are annotated against the curated in-house MS/MS library before unknowns fall back to in silico fragmentation (MS-FINDER) with structure databases.

## Related tools

- **mspcompiler** (R package that reads, structures-annotates (SMILES/RI/MF), polarity-splits, and writes organized msp files from NIST, MoNA, GNPS, RIKEN, SWGDRUG, and in-house libraries for MS-DIAL identification) — https://github.com/QizhiSu/mspcompiler
- **MS-DIAL 4.38** (Performs untargeted DIA feature detection and MS/MS library matching against the compiled in-house msp library on UPLC-QTOF-MS data)
- **Lib2NIST** (Converts installed NIST (and SWGDRUG) libraries into Text File(.MSP) + MOLfiles format suitable for mspcompiler parsing) — https://chemdata.nist.gov/dokuwiki/doku.php?id=chemdata:nist17
- **MS-CleanR** (Cleans MS-DIAL-exported feature tables (blank/QC removal, RSD > 30 discarded) prior to annotation and structural elucidation)
- **MS-FINDER** (Fallback in silico fragmentation for unknowns not annotated by the MS/MS library, using volDB, cppDB, and genDB structure databases)

## Examples

```
library(mspcompiler); in_house <- read_multilibs("D:/MS_libraries/in_house"); combine_ms2_pos <- c(nist_ms2_pos, riken_ms2_pos, mona_ms2_pos, gnps_pos, in_house); write_MS2_msp(combine_ms2_pos, "D:/MS_libraries/combine_ms2_pos.msp")
```

## Evaluation signals

- Feature-annotation counts reconcile: the study annotated 66 of 83 total identified compounds using the in-house MS/MS library and mspcompiler.
- Every library record written to the final msp carries a SMILES (and InChIKey where available), and RI assignments keep only capillary-column records with median RI and discard values with SD > 30.
- Combined pos/neg MS2 msp files parse cleanly in MS-DIAL with no mixed-polarity entries after separate_polarity.
- downstream MRMPROBS pseudo-MRM transitions (top 5 product ions per compound) can be derived from the same verified library, confirming library integrity across 449 positive and 172 negative entries.
- Open library release matches Zenodo deposit (zenodo.org/record/4454648) and GitHub (github.com/QizhiSu/MS-libraries).

## Limitations

- Strategy works only for known unknowns; unknown unknowns absent from all structure databases (volDB, cppDB, genDB) cannot be resolved this way.
- High-molecular-weight substances such as octocrylene and 2-ethylhexyl-4-methoxycinnamate remain hard to identify/removal remains challenging for safe rHDPE food contact.
- Processing NIST EI libraries is time-consuming (several hours) due to large .MSP files and hundreds of thousands of .MOL files; parallel computing is recommended.
- Feature cleaning on DIA data still requires manual spectral examination despite automated filtering.
- GNPS mgf lacks Molecular Formula field; MF must be computed from SMILES, and SWGDRUG lacks InChIKey so match = "name" must be used instead of inchikey.

## Evidence

- [abstract] Curated plastic MS/MS library drives high annotation rate: "A total of 83 compounds were identified, with 66 easily annotated by making use of our in-house MS/MS libraries and the mspcompiler R package."
- [readme] mspcompiler purpose per README: "The goal of mspcompiler is to offer ways to compile either EI or tandem mass spectral libraries from various sources, such as NIST (if you have it installed), MoNA, and GPNS, and organize them into a"
- [readme] RI assignment rules: "When there are multiple records for a single compound, the median RI will be used and if the standard deviation is higher than 30, this value will be discarded."
- [discussion] Known-unknowns limitation: "this strategy only works for known unknowns, not unknown unknowns, which is much more challenging as they might not be present in any existing structure databases"
- [readme] SWGRUG name-only matching caveat: "As the SWGDRUG file does not contain InChIKey information, even though you are working with Linux-based or Mac OS, you should not use "match = inchikey". "match = "name" is more than enough in this"
- [methods] Library scale on GitHub: "which is available at https://github.com/QizhiSu/MS-libraries"
