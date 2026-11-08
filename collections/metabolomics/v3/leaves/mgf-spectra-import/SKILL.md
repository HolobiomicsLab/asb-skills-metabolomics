---
name: mgf-spectra-import
description: Use when MS2 spectra were produced by an external feature-based tool
  (MZmine, MS-DIAL) exported as MGF/MSP files and must be loaded into R as a Spectra
  object for downstream matchSpectra() annotation.
when_to_use_negative:
- Input is already a feature table (CSV) without fragment spectra — use MS1 m/z-based
  matching (matchMz) instead
- Data are raw vendor/mzML files requiring peak detection first — the MGF import path
  assumes spectra already processed by an external tool such as MZmine
- The reference library provides precursor m/z and requirePrecursor = TRUE filtering
  is intended — this skill's example disables precursor filtering because HMDB lacks
  precursor m/z
edam_operation: http://edamontology.org/operation_0256
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MsBackendMgf
  role: Backend used as data source to import MGF-exported spectra files into a Spectra
    object
- name: Spectra
  role: Provides the Spectra container that holds the imported MGF spectra as the
    query for matchSpectra
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MetaboAnnotation
  role: Provides matchSpectra with CompareSpectraParam (ppm = 50, requirePrecursor
    = FALSE) used on the imported query spectra
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: CompoundDb
  role: Supplies the CompDb reference compound database (HMDB 5.0) wrapped into the
    target Spectra object
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: BiocManager
  role: Installs the Spectra/MsBackendMgf/MetaboAnnotation/CompoundDb packages used
    in this workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:5f128c59aa8adb61c06b52fec7054cb21b9264570571474fe2f0ed5f35ddae57
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - CompoundDb
  - MetaboAnnotation
  - MsBackendMgf
  - R
  - Spectra
  - curl
  edam_operation: http://edamontology.org/operation_0256
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mgf-spectra-import

## Summary

Imports externally exported MS2 spectra files (e.g. MZmine or MS-DIAL GNPS-exported MGF) into R as a Spectra object using MsBackendMgf, so they can serve as query spectra for matchSpectra()-based MS2 annotation.

## When to use

Use when MS2 spectra were produced by an external feature-based tool (MZmine, MS-DIAL) exported as MGF/MSP files and must be loaded into R as a Spectra object for downstream matchSpectra() annotation.

## When NOT to use

- Input is already a feature table (CSV) without fragment spectra — use MS1 m/z-based matching (matchMz) instead
- Data are raw vendor/mzML files requiring peak detection first — the MGF import path assumes spectra already processed by an external tool such as MZmine
- The reference library provides precursor m/z and requirePrecursor = TRUE filtering is intended — this skill's example disables precursor filtering because HMDB lacks precursor m/z

## Inputs

- MZmine- or MS-DIAL-exported MGF spectra file (e.g. MZmine-GNPS_AG_test_GNPS.mgf)
- GNPS-exported MGF file from the CCMS-UCSD/GNPSDocumentation repository
- Reference compound database as CompDb SQLite (CompDb.Hsapiens.HMDB.5.0.sqlite)

## Outputs

- Spectra object with MGF metadata variables retained (e.g. FEATURE_ID)
- Match results table (spectraData/matchedData) written to e.g. mzmine_ms2_spectra_matches.csv

## How to apply

Download the MZmine-exported GNPS MGF file (e.g. MZmine-GNPS_AG_test_GNPS.mgf) with curl_download, then construct the query object with Spectra(<mgf path>, source = MsBackendMgf()) from the MsBackendMgf package. Verify that MGF metadata fields such as FEATURE_ID (feature identifiers) are retained in the resulting Spectra object, since these link matches back to the original feature table. Optionally preprocess query spectra before annotation (e.g. filterIntensity to remove peaks below 5% of base peak intensity, drop spectra with fewer than 3 peaks). Build the target reference as a Spectra object (e.g. Spectra(cdb) wrapping the CompDb.Hsapiens.HMDB.5.0.sqlite CompoundDb), then run matchSpectra with CompareSpectraParam(ppm = 50, requirePrecursor = FALSE) — disabling the precursor-m/z filter when the reference database (HMDB) lacks precursor m/z values, with the default similarity threshold of 0.7 (normalized dot product). Extract matches with matchedData selecting columns such as FEATURE_ID, target_compound_id, target_name and score.

## Related tools

- **MsBackendMgf** (Backend used as data source to import MGF-exported spectra files into a Spectra object)
- **Spectra** (Provides the Spectra container that holds the imported MGF spectra as the query for matchSpectra) — https://rformassspectrometry.github.io/Spectra/
- **MetaboAnnotation** (Provides matchSpectra with CompareSpectraParam (ppm = 50, requirePrecursor = FALSE) used on the imported query spectra) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **CompoundDb** (Supplies the CompDb reference compound database (HMDB 5.0) wrapped into the target Spectra object) — https://rformassspectrometry.github.io/CompoundDb/
- **BiocManager** (Installs the Spectra/MsBackendMgf/MetaboAnnotation/CompoundDb packages used in this workflow)

## Examples

```
sps <- Spectra(file.path(tempdir(), s_file), source = MsBackendMgf())
```

## Evaluation signals

- The imported Spectra object retains the FEATURE_ID metadata variable from the original MGF fields
- mzmine_ms2_spectra_matches.csv exists and is parseable
- The match table contains rows with score entries for matched query-target spectrum pairs
- The chosen backend (MsBackendMgf) successfully parsed the file — no empty Spectra object (check length(spectra) > 0)
- CompareSpectraParam used with requirePrecursor = FALSE when the reference database (HMDB) does not provide precursor m/z

## Limitations

- HMDB-based targets have no precursor m/z, so the precursor-m/z filter must be disabled (requirePrecursor = FALSE), which can increase false matches
- MGF export may lose or rename metadata fields; FEATURE_ID retention must be verified after import
- Default similarity threshold is 0.7 (normalized dot product); spectra with fewer than ~3 peaks or low-intensity peaks may need filtering before matching

## Evidence

- [abstract] ecosystem format support: "The ecosystem supports data from a variety of formats, including, but not limited to, MSP, MGF, mzML, mzXML, netCDF as well as MassBank text files and SQL databases."
- [methods] Spectra import from files: "Such `Spectra` objects can be either imported from a variety of input formats, including MGF, MSP, mzML, mzXML or netCDF files"
- [readme] MsBackendMgf installation support: "## Support for MGF files
BiocManager::install("MsBackendMgf")"
- [other] task workflow import call: [evidence span withheld — anchor status: not_found]
