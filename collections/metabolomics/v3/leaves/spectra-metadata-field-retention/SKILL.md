---
name: spectra-metadata-field-retention
description: Use when query spectra come from an external feature-based tool (MZmine,
  MS-DIAL) as an exported MGF file and feature identifiers must survive import for
  later join-back of matchSpectra annotation results.
when_to_use_negative:
- Query spectra are generated in-R from an xcms result via chromPeakSpectra() — metadata
  linkage is handled by xcms, not by MGF field retention
- The external tool exported a feature table or MSP rather than an MGF spectra file
  — MsBackendMgf is not the appropriate source backend
- Precursor m/z filtering is required but the target library lacks precursor m/z (e.g.
  HMDB CompDb), where requirePrecursor must be FALSE regardless of import
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MsBackendMgf
  role: Spectra backend that imports MGF files and retains MGF fields (e.g. FEATURE_ID)
    as spectra metadata variables
- name: Spectra
  role: Provides the Spectra object container used to import the MGF query spectra
    via source = MsBackendMgf()
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MetaboAnnotation
  role: Consumes the imported Spectra object as the query in matchSpectra() with CompareSpectraParam
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: CompoundDb
  role: Provides the CompDb target reference database wrapped in a Spectra object
    for annotation
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: curl
  role: Downloads the MZmine-exported GNPS MGF test file from the GNPSDocumentation
    repository
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:31ac034df5b7ec1caf6beda9f9f27ef3d4f67943993687ded69001a17f9f5734
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# spectra-metadata-field-retention

## Summary

Import externally exported MS2 spectra (e.g. MZmine/MS-DIAL GNPS MGF files) into R as a Spectra object with MsBackendMgf, ensuring that feature-level metadata fields such as FEATURE_ID are retained through import so downstream annotation results can be traced back to features.

## When to use

Use when query spectra come from an external feature-based tool (MZmine, MS-DIAL) as an exported MGF file and feature identifiers must survive import for later join-back of matchSpectra annotation results.

## When NOT to use

- Query spectra are generated in-R from an xcms result via chromPeakSpectra() — metadata linkage is handled by xcms, not by MGF field retention
- The external tool exported a feature table or MSP rather than an MGF spectra file — MsBackendMgf is not the appropriate source backend
- Precursor m/z filtering is required but the target library lacks precursor m/z (e.g. HMDB CompDb), where requirePrecursor must be FALSE regardless of import

## Inputs

- MZmine- or MS-DIAL-exported MGF file of MS2 spectra (e.g. MZmine-GNPS_AG_test_GNPS.mgf)
- MsBackendMgf package (Bioconductor) as the Spectra data source
- Optional: CompDb reference database (CompDb.Hsapiens.HMDB.5.0.sqlite) for subsequent matchSpectra use

## Outputs

- Spectra object retaining external metadata variables such as FEATURE_ID from MGF fields
- Downstream match table (e.g. mzmine_ms2_spectra_matches.csv) with FEATURE_ID, target_compound_id, target_name, score columns

## How to apply

Download the tool-exported MGF (e.g. MZmine-GNPS_AG_test_GNPS.mgf from CCMS-UCSD/GNPSDocumentation via curl_download) and import it with Spectra(file, source = MsBackendMgf()) from the MsBackendMgf package, which parses MGF fields into Spectra metadata variables. Immediately verify that feature-identifier fields (e.g. FEATURE_ID) are present in spectraData(...) — if the exporter dropped or renamed them, the annotation output cannot be linked back to MZmine features. Use the resulting Spectra object directly as the query in matchSpectra() against a target library (e.g. Spectra(cdb) wrapping CompDb.Hsapiens.HMDB.5.0.sqlite with CompareSpectraParam(ppm = 50, requirePrecursor = FALSE), default 0.7 similarity threshold), then extract matchedData columns including the retained FEATURE_ID alongside target_compound_id, target_name and score.

## Related tools

- **MsBackendMgf** (Spectra backend that imports MGF files and retains MGF fields (e.g. FEATURE_ID) as spectra metadata variables)
- **Spectra** (Provides the Spectra object container used to import the MGF query spectra via source = MsBackendMgf()) — https://rformassspectrometry.github.io/Spectra/
- **MetaboAnnotation** (Consumes the imported Spectra object as the query in matchSpectra() with CompareSpectraParam) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **CompoundDb** (Provides the CompDb target reference database wrapped in a Spectra object for annotation) — https://rformassspectrometry.github.io/CompoundDb/
- **curl** (Downloads the MZmine-exported GNPS MGF test file from the GNPSDocumentation repository)

## Examples

```
sps <- Spectra(file.path(tempdir(), "MZmine-GNPS_AG_test_GNPS.mgf"), source = MsBackendMgf())
```

## Evaluation signals

- The imported Spectra object retains the FEATURE_ID metadata variable visible in spectraData()
- mzmine_ms2_spectra_matches.csv exists and is parseable
- The match table contains rows with score entries for matched query-target spectrum pairs
- Each matched row can be joined back to a feature in the original MZmine feature table via FEATURE_ID

## Limitations

- Requires the exporting tool to write feature identifiers into the MGF; fields dropped at export cannot be recovered at import
- The GNPSDocumentation MGF test file is tied to the GNPS export convention — other exporters may use different field names
- matchSpectra against HMDB CompDb must disable the precursor-m/z filter (requirePrecursor = FALSE) since HMDB does not provide precursor m/z

## Evidence

- [methods] MZmine export importable as Spectra with retained FEATURE_ID: [evidence span withheld — anchor status: not_found]
- [abstract] Ecosystem supports MGF and other formats: "The ecosystem supports data from a variety of formats, including, but not limited to, MSP, MGF, mzML, mzXML, netCDF as well as MassBank text files and SQL databases."
- [readme] README installation of MsBackendMgf: "## Support for MGF files
BiocManager::install("MsBackendMgf")"
- [abstract] MS2-based annotation comparing fragment spectra against a reference library: "followed by an MS2-based annotation in which experimental fragment spectra are compared against a reference library."
