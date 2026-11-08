---
name: matched-spectra-result-extraction
description: Use when a matchSpectra() MatchedSpectra object (query spectra imported
  e.g. from MZmine MGF via MsBackendMgf against a CompDb/HMDB target) must be converted
  into a flat per-feature annotation table with compound names and scores.
when_to_use_negative:
- When the query spectra come from an xcms result object processed within the same
  R session rather than an external MZmine/MS-DIAL MGF export — use chromPeakSpectra/whichQuery
  on the xcms object instead.
- When only MS1-level annotation is needed (m/z or m/z+RT matching with matchMz/Mass2MzParam)
  — no MS2 spectra matching or match-result extraction applies.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotation
  role: Provides matchSpectra() to compare query spectra against a reference library
    and matchedData/spectraData to extract match results
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: Spectra
  role: Spectra object container for imported MGF query spectra and CompDb-wrapped
    target spectra
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MsBackendMgf
  role: Backend used to import MZmine-exported GNPS MGF files as Spectra, retaining
    FEATURE_ID metadata
- name: CompoundDb
  role: Provides the CompDb reference compound database (HMDB 5.0) used to build target
    spectra
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: R
  role: Language in which the full annotation and result-extraction workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:83cd1e8334ebed5981574c5aab1daeb5622b26bec21f748f790cbb0d0d55ed5f
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# matched-spectra-result-extraction

## Summary

Extracts MS2 spectrum match results from a MetaboAnnotation matchSpectra result object into a tabular annotation table (e.g. via spectraData/matchedData) and writes it to CSV, preserving query feature identifiers alongside target compound metadata and similarity scores.

## When to use

Use when a matchSpectra() MatchedSpectra object (query spectra imported e.g. from MZmine MGF via MsBackendMgf against a CompDb/HMDB target) must be converted into a flat per-feature annotation table with compound names and scores.

## When NOT to use

- When the query spectra come from an xcms result object processed within the same R session rather than an external MZmine/MS-DIAL MGF export — use chromPeakSpectra/whichQuery on the xcms object instead.
- When only MS1-level annotation is needed (m/z or m/z+RT matching with matchMz/Mass2MzParam) — no MS2 spectra matching or match-result extraction applies.

## Inputs

- MatchedSpectra object returned by matchSpectra() from MetaboAnnotation
- Query Spectra object imported from an MZmine-exported MGF file (e.g. MZmine-GNPS_AG_test_GNPS.mgf) with MsBackendMgf, retaining FEATURE_ID metadata
- Target Spectra object wrapping a CompDb database (CompDb.Hsapiens.HMDB.5.0.sqlite)

## Outputs

- mzmine_ms2_spectra_matches.csv containing per-feature match rows with FEATURE_ID, target_compound_id, target_name and score
- Match table (data.frame) of matched query-target spectrum pairs with normalized dot-product scores

## How to apply

After running matchSpectra(std_spectra, Spectra(cdb), param = CompareSpectraParam(ppm = 50, requirePrecursor = FALSE)) with the default 0.7 normalized-dot-product threshold, extract the result table with spectraData or matchedData, selecting relevant columns such as FEATURE_ID (retained from the MGF fields of the MZmine export), target_compound_id, target_name and score. Write the extracted table to mzmine_ms2_spectra_matches.csv. Verify that the imported Spectra object retained the FEATURE_ID metadata variable and that match rows carry score entries for matched query-target pairs; disable the precursor-m/z filter (requirePrecursor = FALSE) when the target database (HMDB) does not provide precursor m/z values.

## Related tools

- **MetaboAnnotation** (Provides matchSpectra() to compare query spectra against a reference library and matchedData/spectraData to extract match results) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **Spectra** (Spectra object container for imported MGF query spectra and CompDb-wrapped target spectra) — https://rformassspectrometry.github.io/Spectra/
- **MsBackendMgf** (Backend used to import MZmine-exported GNPS MGF files as Spectra, retaining FEATURE_ID metadata)
- **CompoundDb** (Provides the CompDb reference compound database (HMDB 5.0) used to build target spectra) — https://rformassspectrometry.github.io/CompoundDb/
- **R** (Language in which the full annotation and result-extraction workflow is executed)

## Examples

```
mtches <- matchSpectra(std_spectra, Spectra(cdb), param = CompareSpectraParam(ppm = 50, requirePrecursor = FALSE)); res <- matchedData(mtches, c("FEATURE_ID", "target_compound_id", "target_name", "score")); write.csv(res, "mzmine_ms2_spectra_matches.csv", row.names = FALSE)
```

## Evaluation signals

- mzmine_ms2_spectra_matches.csv exists and is parseable
- The imported Spectra object retains the FEATURE_ID metadata variable from the original MGF fields
- The match table contains rows with non-empty score entries for matched query-target spectrum pairs
- Selected output columns (FEATURE_ID, target_compound_id, target_name, score) are all present in the written CSV
- Scores correspond to the default 0.7 normalized dot-product threshold used by CompareSpectraParam

## Limitations

- HMDB CompDb targets do not provide precursor m/z, so requirePrecursor must be set to FALSE, which can increase false-positive matches above the 0.7 similarity threshold
- Metadata retention (e.g. FEATURE_ID) depends on fields present in the exported MGF; MZmine or MS-DIAL exports missing FEATURE_ID cannot be traced back to features
- Default threshold of 0.7 (normalized dot product) may report multiple candidate compounds per spectrum; most peaks can match several compounds based on m/z alone

## Evidence

- [methods] MZmine (or MS-DIAL) exported spectra files can be imported directly into R as a Spectra object using MsBackendMgf as the data source, making them usable as the query in matchSpectra(): [evidence span withheld — anchor status: not_found]
- [methods] CompareSpectraParam with requirePrecursor disabled for HMDB targets: [evidence span withheld — anchor status: not_found]
- [methods] Extraction of matched columns to CSV: [evidence span withheld — anchor status: not_found]
- [methods] Spectra import supports MGF among other formats: "Such `Spectra` objects can be either imported from a variety of input formats, including MGF, MSP, mzML, mzXML or netCDF files"
- [readme] Tutorial repository availability in README: "This repository provides use cases and examples for the annotation of non-target
metabolomics or small compound MS data using the `MetaboAnnotation` and
`MetaboCoreUtils` R packages."
