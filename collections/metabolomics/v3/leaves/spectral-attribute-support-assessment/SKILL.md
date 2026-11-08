---
name: spectral-attribute-support-assessment
description: Use when you must decide which R package to adopt for reading or writing
  mass-spectral libraries in NIST msp / MoNA msp / GNPS mgf dialects, or when compiling
  a package capability matrix for spectral-file format support.
when_to_use_negative:
- The input is already a parsed, in-memory spectral data structure (e.g. an R Spectra/MSnbase
  object) rather than a serialized msp or mgf file
- The task concerns raw chromatogram-mass-spectrometry file I/O (e.g. mzML, NetCDF,
  CDF) rather than spectral-library exchange formats
- Only a single package's parser is of interest and no cross-package comparison matrix
  is needed
edam_operation: http://edamontology.org/operation_3351
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Programming environment in which the reader/writer functions of the reviewed
    packages are benchmarked against the reference msp/mgf dialects
- name: metaMS
  role: Reviewed package supporting export to NIST msp format; its export step is
    timed (~20 min for an 8 MB file) as part of the benchmark
- name: OrgMassSpecR
  role: Reviewed package supporting export to the NIST msp file format, included in
    the capability matrix
- name: enviGCMS
  role: Reviewed package supporting export to the NIST msp file format, included in
    the capability matrix
- name: TargetSearch
  role: Reviewed package supporting export to the NIST msp file format, included in
    the capability matrix
- name: MSnbase
  role: Reviewed package whose msp/mgf reader functions are benchmarked on the reference
    dialects
- name: compMS2Miner
  role: Reviewed package whose spectral-library reader functions are benchmarked on
    the reference dialects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:7a78d140a57e58b97d8a525beedaf6019782c000fc843e689ee4e755b65d99c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - OrgMassSpecR
  - R
  - TargetSearch
  - compMS2Miner
  - enviGCMS
  - metaMS
  edam_operation: http://edamontology.org/operation_3351
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spectral-attribute-support-assessment

## Summary

A benchmarking method that determines which R packages (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner) can read and write spectral-library exchange formats such as NIST msp, MoNA msp, and GNPS mgf, and which spectral attributes (e.g. InChIKey, Collision_energy) survive import/export round-trips. It surfaced from the metaRbolomics review's comparison of spectral-database readers and writers (Table 3, Table S1).

## When to use

Use when you must decide which R package to adopt for reading or writing mass-spectral libraries in NIST msp / MoNA msp / GNPS mgf dialects, or when compiling a package capability matrix for spectral-file format support.

## When NOT to use

- The input is already a parsed, in-memory spectral data structure (e.g. an R Spectra/MSnbase object) rather than a serialized msp or mgf file
- The task concerns raw chromatogram-mass-spectrometry file I/O (e.g. mzML, NetCDF, CDF) rather than spectral-library exchange formats
- Only a single package's parser is of interest and no cross-package comparison matrix is needed

## Inputs

- Reference spectral-library snippets of Supplemental File S1 (Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME')
- Public MoNA msp export files
- Public GNPS mgf export files
- Reader and writer functions from the R packages metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, and compMS2Miner

## Outputs

- Package x dialect/attribute capability matrix as CSV with boolean/enumerated support entries, one row per package
- Per-package parse success/failure records for each msp/mgf dialect
- Round-trip attribute retention records (e.g. InChIKey, Collision_energy)
- Measured metaMS export runtime (~20 min for an 8 MB file)

## How to apply

Parse reference spectral-library snippets (Listing S1 'basic NIST', S2 'canonical NIST', S3 'RIKEN PRIME' from Supplemental File S1) with the reader functions of each R package and record parse success/failure per package per dialect. Repeat the read benchmark on public MoNA msp and GNPS mgf exports to cover the two public exchange dialects. Then run each package's writer functions on the parsed spectra and record which dialect and which attributes (e.g. InChIKey, Collision_energy) survive the import/export round-trip. Time the metaMS export step separately, since the review reports a runtime of about 20 minutes for an 8 MB file. Assemble all results into a package x dialect/attribute capability matrix as CSV with boolean/enumerated support entries, one row per package.

## Related tools

- **R** (Programming environment in which the reader/writer functions of the reviewed packages are benchmarked against the reference msp/mgf dialects)
- **metaMS** (Reviewed package supporting export to NIST msp format; its export step is timed (~20 min for an 8 MB file) as part of the benchmark)
- **OrgMassSpecR** (Reviewed package supporting export to the NIST msp file format, included in the capability matrix)
- **enviGCMS** (Reviewed package supporting export to the NIST msp file format, included in the capability matrix)
- **TargetSearch** (Reviewed package supporting export to the NIST msp file format, included in the capability matrix)
- **MSnbase** (Reviewed package whose msp/mgf reader functions are benchmarked on the reference dialects)
- **compMS2Miner** (Reviewed package whose spectral-library reader functions are benchmarked on the reference dialects)

## Examples

```
pkgs <- c('metaMS','OrgMassSpecR','enviGCMS','TargetSearch'); sapply(pkgs, function(p) writeMsp(readMsp('Listing_S1_basic_NIST.msp'), paste0(p, '_roundtrip.msp')))
```

## Evaluation signals

- The CSV capability matrix has exactly one row per reviewed package (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner)
- The matrix contains dialect and attribute columns, and all support entries are valid boolean/enumerated values
- metaMS, OrgMassSpecR, enviGCMS, and TargetSearch are flagged as the packages supporting export to the NIST msp format
- The recorded metaMS export runtime is in the order of magnitude of ~20 minutes for an 8 MB file
- Round-trip records specify per attribute (e.g. InChIKey, Collision_energy) whether it survives import/export for each package

## Limitations

- metaMS export to NIST msp is slow: ~20 minutes for an 8 MB reference file, which may be prohibitive for large libraries
- Attribute retention differs across packages and dialects — some attributes such as InChIKey or Collision_energy may not survive import/export round-trips
- Coverage is limited to the reviewed R packages and the msp/mgf dialects tested (basic NIST, canonical NIST, RIKEN PRIME, MoNA msp, GNPS mgf); other formats and tools are out of scope

## Evidence

- [supplementary] metaMS, OrgMassSpecR, enviGCMS, and TargetSearch are the reviewed R packages that support export to the NIST msp file format: [evidence span withheld — anchor status: not_found]
- [abstract] The programming and statistics environment R has emerged as one of the most popular environments: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [abstract] In total, this review summarises more than two hundred metabolomics specific packages: "In total, this review summarises more than two hundred metabolomics specific packages primarily available on CRAN, Bioconductor and GitHub."
- [other] Time the metaMS export step on the reference content and record the measured runtime: [evidence span withheld — anchor status: not_found]
