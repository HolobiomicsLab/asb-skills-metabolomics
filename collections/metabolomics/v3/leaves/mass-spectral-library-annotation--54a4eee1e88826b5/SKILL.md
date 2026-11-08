---
name: mass-spectral-library-annotation
description: Use when untargeted LC-QTOF-MS/MS screening data (e.g. MS-DIAL-exported
  feature tables and .msp/.mgf spectra) must be annotated against MS/MS libraries
  plus plastic-related and GC–MS-derived structure databases to yield identified compounds
  with library vs unknown status.
when_to_use_negative:
- The analytes of interest are volatile/semi-volatile and already covered by EI-MS
  spectral matching (GC–MS), rather than non-volatile migrants needing LC-HRMS/MS
  annotation.
- Features have already been annotated — e.g. you hold a final identified-compound
  table — and only quantification or pseudo-MRM targeting remains.
- The unknowns are 'unknown unknowns' that may not be present in any existing structure
  database; the article notes in silico fragmentation only works for known unknowns.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS-DIAL 4.38
  role: Deconvolutes and aligns UPLC-QTOF-MS/MS features and performs MS/MS spectral
    matching against the compiled msp libraries to produce library-annotated identifications.
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
- name: mspcompiler
  role: R package that reads, cleans, and compiles NIST/RIKEN/MoNA/GNPS and in-house
    MS/MS libraries into neat msp files usable by MS-DIAL.
  repo: https://github.com/QizhiSu/mspcompiler
- name: MS-CleanR
  role: Removes blank/QC features and features with RSD above 0.5 and 30 from the
    MS-DIAL feature table before annotation.
- name: MS-FINDER
  role: Runs in silico fragmentation on features unmatched by the MS/MS libraries
    against cppDB, volDB, and genDB structure databases to identify unknowns.
- name: Lib2NIST
  role: Converts installed NIST (and SWGDRUG NIST-format) libraries into msp plus
    MOL files that mspcompiler can process.
  repo: https://chemdata.nist.gov/dokuwiki/doku.php?id=chemdata:nist17
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:bca5bde3aa48814803877de0a352d52b05bae990a86bd313545eb8e4bf346cfe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lib2NIST
  - MS-CleanR
  - MS-DIAL
  - MS-DIAL 4.38
  - MS-FINDER
  - mspcompiler
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# mass-spectral-library-annotation

## Summary

Annotate LC-HRMS features of non-volatile migrants by matching processed MS/MS spectra against compiled in-house and public tandem libraries within MS-DIAL, falling back to in silico fragmentation (MS-FINDER) against structure databases for remaining unknowns.

## When to use

Use when untargeted LC-QTOF-MS/MS screening data (e.g. MS-DIAL-exported feature tables and .msp/.mgf spectra) must be annotated against MS/MS libraries plus plastic-related and GC–MS-derived structure databases to yield identified compounds with library vs unknown status.

## When NOT to use

- The analytes of interest are volatile/semi-volatile and already covered by EI-MS spectral matching (GC–MS), rather than non-volatile migrants needing LC-HRMS/MS annotation.
- Features have already been annotated — e.g. you hold a final identified-compound table — and only quantification or pseudo-MRM targeting remains.
- The unknowns are 'unknown unknowns' that may not be present in any existing structure database; the article notes in silico fragmentation only works for known unknowns.

## Inputs

- LC-QTOF-MS/MS raw data from UPLC-QTOF-MS analysis of solid–liquid extracts and migration-test samples (e.g. rHDPE milk bottles)
- MS-DIAL version 4.38 output: feature table and exported MS/MS spectra (msp/mgf)
- In-house MS/MS library containing more than 500 plastic-related chemicals
- Public tandem libraries: NIST MS2 msp + MOL files, RIKEN MS/MS-Public-Pos/Neg msp, MoNA LC-MS/MS Positive/Negative msp, GNPS ALL_GNPS.mgf
- Structure databases: cppDB (plastic packaging chemicals), volDB (GC–MS compounds from the same samples), genDB integrated in MS-FINDER (FoodDB, PlantCyc, T3DB, STOFF, NPA, KNApSAcK)
- Experimental retention index files ri.dat and USER.DBU (for EI-library RI annotation)

## Outputs

- List of library-annotated compounds (e.g. 66 of 83 in the rHDPE study) from MS-DIAL MS/MS library matching
- List of unknown compounds identified only via MS-FINDER in silico fragmentation against structure databases (e.g. 13 of 83), with spectra retained in a supplemental appendix
- Combined, cleaned positive and negative MS/MS msp library files (combine_ms2_pos.msp, combine_ms2_neg.msp)
- Identification tally table partitioning compounds by identification status (library-annotated vs unknown)
- Pseudo-multiple reaction monitoring targets derived from annotated compounds for sensitive screening

## How to apply

Process LC-QTOF-MS/MS raw data (Waters Acquity UPLC with Atlantis premier BEH C18 AX column, ESI) through MS-DIAL version 4.38, then clean the exported feature table with MS-CleanR by removing blank/QC features and features with relative standard deviations higher than 0.5 and 30, respectively. Compile a tandem library for matching using mspcompiler: read public libraries (NIST MS2 split by separate_polarity, RIKEN MS/MS Positive/Neg msp, MoNA LC-MS/MS with reorganize_mona to move SMILES out of the Comment field, GNPS mgf with complete_mgf to compute molecular formula from SMILES) plus the in-house library of more than 500 plastic-related chemicals, then write combined positive and negative msp files via write_MS2_msp. Match features against these libraries in MS-DIAL to obtain library-annotated identifications; keep both the most intense and most connected features for subsequent structural elucidation in MS-FINDER. For features that remain unannotated, run MS-FINDER in silico fragmentation against cppDB (chemicals associated with plastic packaging), volDB (GC–MS-identified substances from the same samples), and the genDB integrated in MS-FINDER (FoodDB, PlantCyc, T3DB, STOFF, NPA, KNApSAcK). Retain spectra of identified-but-unannotated unknowns in a supplemental appendix for further exploration, and tally results as total identified = library-annotated + unknown (in the rHDPE milk-bottle case: 83 total, 66 library-annotated, 13 unknown).

## Related tools

- **MS-DIAL 4.38** (Deconvolutes and aligns UPLC-QTOF-MS/MS features and performs MS/MS spectral matching against the compiled msp libraries to produce library-annotated identifications.) — http://prime.psc.riken.jp/compms/msdial/main.html
- **mspcompiler** (R package that reads, cleans, and compiles NIST/RIKEN/MoNA/GNPS and in-house MS/MS libraries into neat msp files usable by MS-DIAL.) — https://github.com/QizhiSu/mspcompiler
- **MS-CleanR** (Removes blank/QC features and features with RSD above 0.5 and 30 from the MS-DIAL feature table before annotation.)
- **MS-FINDER** (Runs in silico fragmentation on features unmatched by the MS/MS libraries against cppDB, volDB, and genDB structure databases to identify unknowns.)
- **Lib2NIST** (Converts installed NIST (and SWGDRUG NIST-format) libraries into msp plus MOL files that mspcompiler can process.) — https://chemdata.nist.gov/dokuwiki/doku.php?id=chemdata:nist17

## Examples

```
gnps <- read_lib("D:/MS_libraries/ALL_GNPS.mgf", format = "mgf"); gnps <- complete_mgf(gnps); gnps_pos <- separate_polarity(gnps, polarity = "pos")
```

## Evaluation signals

- Total identifications equal the sum of library-annotated and unknown counts (rHDPE study: 83 = 66 + 13); check the partition is mutually exclusive and exhaustive.
- Combined MS/MS msp files for positive and negative modes parse cleanly in MS-DIAL and contain no entries missing SMILES or molecular formula fields (GNPS entries must have MF computed by complete_mgf).
- Feature table after MS-CleanR contains no blank/QC features and no features with relative standard deviation above 30.
- For each library-annotated compound, the matched MS/MS spectrum and retention behavior are consistent with the corresponding in-house or public library entry; cross-check a subset against the in-house >500-compound plastic library.
- Each unknown identification is backed by an MS-FINDER formula/score against a named structure database (cppDB, volDB, or genDB), and its spectrum is archived in the supplemental appendix for further exploration.

## Limitations

- The strategy only works for known unknowns, not unknown unknowns, which may not be present in any existing structure database.
- Manual examination of the spectra is necessary during feature cleaning when dealing with data independent acquisition data.
- High-molecular weight substances such as octocrylene remain obstacles for food contact applications of rHDPE because they are not easy to remove.
- Processing the NIST EI msp + MOL folder through mspcompiler is time-consuming (several hours depending on PC capability); parallel computing is recommended.
- On Linux/Mac assign_smiles should use match = "inchikey", but libraries lacking InChIKey (e.g. SWGDRUG) must still use match = "name".

## Evidence

- [abstract] A total of 83 compounds were identified, with 66 easily annotated by making use of our in-house MS/MS libraries and the mspcompiler R package.: "A total of 83 compounds were identified, with 66 easily annotated by making use of our in-house MS/MS libraries and the mspcompiler R package."
- [methods] Feature tables exported from MS-DIAL were cleaned by removing those with blank/QC and relative standard deviations higher than 0.5 and 30, respectively, using the MS-CleanR package.: "Feature tables exported from MS-DIAL were cleaned by removing those with blank/QC and relative standard deviations higher than 0.5 and 30, respectively, using the MS-CleanR package."
- [abstract] In silico fragmentation combined with data obtained from gas chromatography–mass spectrometry and lists of chemicals related to plastics were used to identify those remaining unknown.: "In silico fragmentation combined with data obtained from gas chromatography–mass spectrometry and lists of chemicals related to plastics were used to identify those remaining unknown."
- [methods] For subsequent structural elucidation in MS-FINDER, both the most intense and most connected features were kept.: [evidence span withheld — anchor status: not_found]
- [readme] The goal of mspcompiler is to offer ways to compile either EI or tandem mass spectral libraries from various sources, such as NIST (if you have it installed), MoNA, and GPNS, and organize them into a neat and up-to-date msp file that can be used in MS-DIAL.: "The goal of mspcompiler is to offer ways to compile either EI or tandem mass spectral libraries from various sources, such as NIST (if you have it installed), MoNA, and GPNS, and organize them into a"
- [readme] The SMILES has to be extracted from the Comment and put into the SMILES field by the reorganize_mona function.: "it is in the *Comment* field. Therefore, the SMILES has to be extracted from the *Comment* and put into the *SMILES* field by the *reorganize_mona* function."
- [discussion] this strategy only works for known unknowns, not unknown unknowns, which is much more challenging as they might not be present in any existing structure databases: "this strategy only works for known unknowns, not unknown unknowns, which is much more challenging as they might not be present in any existing structure databases"
- [discussion] Manual examination of the spectra is necessary during feature cleaning when dealing with data independent acquisition data: "Manual examination of the spectra is necessary during feature cleaning when dealing with data independent acquisition data"
