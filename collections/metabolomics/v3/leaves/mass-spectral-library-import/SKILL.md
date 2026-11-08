---
name: mass-spectral-library-import
description: Use when deconvolved and aligned GC–MS compounds in a MetaboSet object
  need putative metabolite identities via spectral library matching, and the reference
  library must be loaded or swapped before running identifyComp.
when_to_use_negative:
- Input is a NIST-format library — eRah cannot read it directly since 'NIST library
  can only be read with NIST software'; convert or export spectra to MSP first.
- The aim is targeted analysis by GC-QqQ MS, LC-QqQ, or NMR where the metabolite identities
  are already known and no spectral library matching is needed.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: erah
  role: R package providing importMSP/importGMD to load reference spectral libraries
    and identifyComp/idList to match aligned compound spectra against them
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Language in which the library import and matching functions are executed
- name: Golm Metabolome Database (GMD)
  role: Recommended publicly available reference library for putative metabolite identification
- name: MassBank
  role: Built-in custom mass-spectral library shipped with eRah (~500 unique EI GC–TOF
    spectra)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:509acd0b8156dd7869eb17707f9d0e649ef367dab4259e47c02fa52cc0507635
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Golm Metabolome Database (GMD)
  - MassBank
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mass-spectral-library-import

## Summary

Import a compound reference mass-spectral library (e.g. the Golm Metabolome Database, MassBank, Fiehn, HMDB, or an internal database) into the eRah MetaboSet experiment so that aligned deconvolved compounds can be matched to candidate metabolites with similarity scores.

## When to use

Use when deconvolved and aligned GC–MS compounds in a MetaboSet object need putative metabolite identities via spectral library matching, and the reference library must be loaded or swapped before running identifyComp.

## When NOT to use

- Input is a NIST-format library — eRah cannot read it directly since 'NIST library can only be read with NIST software'; convert or export spectra to MSP first.
- The aim is targeted analysis by GC-QqQ MS, LC-QqQ, or NMR where the metabolite identities are already known and no spectral library matching is needed.

## Inputs

- MetaboSet experiment object (e.g. testPCOS.rda) from prior eRah deconvolution/alignment/recovery steps
- Reference mass-spectral library in MSP format (e.g. GMD_20111121_VAR5_ALK_MSP.txt from the Golm Metabolome Database)

## Outputs

- Populated mslib spectral library object attached to the MetaboSet, used as the reference for subsequent identification
- List of candidate metabolites with similarity match factors per aligned compound, retrieved via idList(ex)

## How to apply

Load the MetaboSet experiment saved after deconvolution, alignment, and any missing-compound recovery into R. Create or replace the default mslib object by importing an MSP-format library: the article recommends the Golm Metabolome Database (GMD), loaded via importGMD(filename='GMD_20111121_VAR5_ALK_MSP.txt', DB.name='GMD', DB.version='GMD_20111121', type='VAR5.ALK'), and notes MassBank (~500 unique EI GC–TOF spectra) ships as the built-in default while NIST libraries cannot be imported because they are only readable with NIST software. The library import must happen before identifyComp so that the mean deconvolved compound spectrum of each AlignID is compared (cosine product) against reference spectra over m/z 70–600 with trimethylsilyl fragments 73–75 and 147–149 excluded. Re-import or re-run identification after recMissComp, since recovered compounds must be re-identified. Identification matches ≥ 80.0 similarity (the paper's threshold for high-quality matches in GMD and MassBank) can then be retrieved with idList().

## Related tools

- **erah** (R package providing importMSP/importGMD to load reference spectral libraries and identifyComp/idList to match aligned compound spectra against them) — http://CRAN.R-project.org/package=erah
- **R** (Language in which the library import and matching functions are executed)
- **Golm Metabolome Database (GMD)** (Recommended publicly available reference library for putative metabolite identification)
- **MassBank** (Built-in custom mass-spectral library shipped with eRah (~500 unique EI GC–TOF spectra))

## Examples

```
ex <- importGMD(filename='GMD_20111121_VAR5_ALK_MSP.txt', DB.name='GMD', DB.version='GMD_20111121', type='VAR5.ALK'); idList(ex)
```

## Evaluation signals

- idList(ex) returns non-empty rows with compound names, retention times, AlignID, and match factors, confirming the library was loaded and queried successfully.
- A majority of retained identifications show similarity match factors > 80.0, the paper's threshold for high-quality matches to GMD/MassBank reference spectra.
- Identification is re-run after recMissComp so recovered compounds also carry match factors (i.e. no AlignIDs present in the alignment list are absent from idList output).
- The mslib object reflects the imported library (e.g. GMD rather than the default MassBank) so identifyComp searches against the intended reference set.

## Limitations

- The NIST library cannot be imported because it is only readable with NIST software; eRah can export found spectra to MSP format but has no NIST importer.
- Imported library must be in a non-binary, text (MSP) readable format with compound EI spectra.
- Missing-compound recovery invalidates previous identifications — compounds must always be re-identified after recMissComp.

## Evidence

- [methods] Import GMD library: "Then, we can load the library with the function importMSP()"
- [results] Identification with match factors: "These compounds showed a high similarity match factor (>80.0) to reference MS spectra in the GMD and Mass Bank."
- [results] Alternative libraries: "users may import other libraries such as the Golm Metabolome Database (GMD),40,41 Fiehn,28 Human Metabolome Database (HMDB)42 or an internal database"
- [results] NIST limitation: "Note that NIST library can only be read with NIST software."
- [methods] Re-identify after recovery: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [methods] Custom MassBank library: "This package includes a custom version of the MassBank MS library"
- [readme] Output table with match scores: "Outputs a table with compound names, matching scores and the integrated area of the compound for each sample."
