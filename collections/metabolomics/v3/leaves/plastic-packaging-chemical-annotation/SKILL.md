---
name: plastic-packaging-chemical-annotation
description: Use when untargeted LC-QTOF-MS feature lists from plastic packaging (e.g.
  rHDPE milk bottles) need compound-level annotation with MS/MS libraries, structure
  databases, and in silico fragmentation.
when_to_use_negative:
- Input is already a curated, library-annotated compound table — no MS-FINDER/cppDB
  step is needed.
- The analytes are volatile/semi-volatile; GC–MS-based workflows (volDB source) are
  the appropriate primary method, not LC-QTOF annotation.
- Unknowns that are 'unknown unknowns' absent from all structure databases — the article
  explicitly states this strategy only works for known unknowns.
- Data come from non-plastic matrices (e.g. natural product extracts); cppDB's plastic-chemical
  coverage is the rationale, so annotation quality degrades.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mspcompiler
  role: R package that cleans and compiles NIST/MoNA/GNPS/RIKEN libraries into msp
    files usable in MS-DIAL; used to build the in-house MS/MS library of >500 plastic-related
    chemicals
  repo: https://github.com/QizhiSu/mspcompiler
- name: MS-DIAL 4.38
  role: Processes untargeted UPLC-QTOF-MS data and performs library matching against
    the compiled msp; also enables pseudo-MRM targeting of identified chemicals
- name: MS-FINDER
  role: In silico fragmentation engine used to elucidate the 13 compounds not covered
    by the MS/MS library, querying cppDB/volDB/genDB structure databases
- name: MS-CleanR
  role: Cleans MS-DIAL feature tables by removing blank/QC features and those with
    RSD > 30% before annotation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:e2f9603e7f3c1129a3c5af8e5125b78b23cda21a106b2aa850a16eb4c8e1315f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - MS-CleanR
  - MS-DIAL
  - MS-DIAL 4.38
  - MS-FINDER
  - mspcompiler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# plastic-packaging-chemical-annotation

## Summary

Annotates non-volatile migrants in recycled HDPE by combining an in-house MS/MS library (mspcompiler), in silico fragmentation (MS-FINDER), GC–MS-derived compound lists (volDB) and the plastic-chemicals database cppDB, yielding 83 identified compounds with 66 library-annotated and 13 unknowns.

## When to use

Use when untargeted LC-QTOF-MS feature lists from plastic packaging (e.g. rHDPE milk bottles) need compound-level annotation with MS/MS libraries, structure databases, and in silico fragmentation.

## When NOT to use

- Input is already a curated, library-annotated compound table — no MS-FINDER/cppDB step is needed.
- The analytes are volatile/semi-volatile; GC–MS-based workflows (volDB source) are the appropriate primary method, not LC-QTOF annotation.
- Unknowns that are 'unknown unknowns' absent from all structure databases — the article explicitly states this strategy only works for known unknowns.
- Data come from non-plastic matrices (e.g. natural product extracts); cppDB's plastic-chemical coverage is the rationale, so annotation quality degrades.

## Inputs

- UPLC-QTOF-MS raw data (positive/negative ESI) from plastic-packaging extracts
- MS-DIAL feature tables (mz, RT, intensity)
- in-house MS/MS library msp (e.g. >500 plastic-related chemicals, mspcompiler-compiled)
- structure databases: cppDB, volDB, genDB (FoodDB, PlantCyc, T3DB, STOFF, NPA, KNApSAcK)
- GC–MS volatile/semi-volatile compound lists from the same samples

## Outputs

- annotated compound list with library-matched vs MS-FINDER/in silico assignments
- tally of identified compounds (e.g. 83 total, 66 library-annotated, 13 unknown) per sample set
- msp-compiled in-house MS/MS library reusable in MS-DIAL
- quantification / pseudo-MRM screening table for identified migrants

## How to apply

Process UPLC-QTOF-MS data with MS-DIAL 4.38, export feature tables, and remove blank/QC features and features with RSD > 30% using MS-CleanR (manual spectrum inspection is still needed for DIA data). For library matching, compile an in-house MS/MS library of >500 plastic-related chemicals into an msp file with the mspcompiler R package and identify compounds directly in MS-DIAL. For remaining unknowns, run in silico fragmentation in MS-FINDER against cppDB (plastic-packaging chemicals), volDB (GC–MS-identified volatiles/semi-volatiles from the same samples), and genDB; keep both the most intense and most connected features for structural elucidation. Finally tally annotations: the article reports 83 identified compounds, 66 annotated via the in-house MS/MS library, and 13 identified only via MS-FINDER/GC–MS/plastic-chemical lists.

## Related tools

- **mspcompiler** (R package that cleans and compiles NIST/MoNA/GNPS/RIKEN libraries into msp files usable in MS-DIAL; used to build the in-house MS/MS library of >500 plastic-related chemicals) — https://github.com/QizhiSu/mspcompiler
- **MS-DIAL 4.38** (Processes untargeted UPLC-QTOF-MS data and performs library matching against the compiled msp; also enables pseudo-MRM targeting of identified chemicals)
- **MS-FINDER** (In silico fragmentation engine used to elucidate the 13 compounds not covered by the MS/MS library, querying cppDB/volDB/genDB structure databases)
- **MS-CleanR** (Cleans MS-DIAL feature tables by removing blank/QC features and those with RSD > 30% before annotation)

## Examples

```
nist_ei <- read_lib("D:/MS_libraries/NIST.MSP", type = "EI"); combine_ei <- assign_ri(combine_ei, nist_ri, polarity = "semi-polar"); write_EI_msp(combine_ei, "D:/MS_libraries/combine_ei.msp")
```

## Evaluation signals

- Tally check: number of identified compounds equals library-annotated count plus MS-FINDER/unknown count (83 = 66 + 13 in the reference run).
- Every library-annotated compound matches an entry in the mspcompiler-compiled in-house MS/MS library of >500 plastic-related chemicals.
- Unknown-identified compounds have MS-FINDER assignments traceable to cppDB, volDB, or genDB structures.
- Feature table has passed MS-CleanR filters: blank/QC features removed and RSD threshold of 30% enforced.
- Spectra of unidentified compounds are retained (e.g. in an appendix such as Appendix C) for further exploration.

## Limitations

- Strategy only works for 'known unknowns'; unknown unknowns absent from existing structure databases cannot be resolved this way.
- Manual examination of spectra is necessary during feature cleaning when dealing with data independent acquisition data.
- Assign_ri discards median RI values when the standard deviation across records exceeds 30, which can leave compounds without RI.
- NIST MoNA/SWGDRUG SMILES extraction depends on correct field handling (Comment field for MoNA; no InChIKey for SWGDRUG, so match='name' must be used).

## Evidence

- [abstract] Total compounds identified: "A total of 83 compounds were identified, with 66 easily annotated by making use of our in-house MS/MS libraries and the mspcompiler R package."
- [methods] Cleaning thresholds: "Feature tables exported from MS-DIAL were cleaned by removing those with blank/QC and relative standard deviations higher than 0.5 and 30, respectively, using the MS-CleanR package."
- [methods] MS-FINDER feature selection: [evidence span withheld — anchor status: not_found]
- [discussion] Strategy limitation: "this strategy only works for known unknowns, not unknown unknowns, which is much more challenging as they might not be present in any existing structure databases"
- [readme] mspcompiler purpose: [evidence span withheld — anchor status: not_found]
