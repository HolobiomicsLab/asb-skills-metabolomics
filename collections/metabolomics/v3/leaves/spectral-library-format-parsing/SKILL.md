---
name: spectral-library-format-parsing
description: Use when R code must parse or export spectral libraries in NIST msp,
  MoNA msp, or GNPS mgf dialects, or when comparing which R packages can read/write
  specific spectral-database attributes.
when_to_use_negative:
- Input is already a parsed in-memory spectral data structure (e.g. an R Spectra/MSnbase
  object) rather than an msp/mgf file on disk
- The goal is general metabolomics data processing (peak picking, alignment, biostatistics)
  rather than spectral-library file interchange between R packages
- Working outside R with tools that natively handle all msp/mgf dialects — the capability
  matrix is specific to the reviewed R packages on CRAN/Bioconductor/GitHub
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metaMS
  role: Reader and writer for NIST msp dialects; its export step is the benchmarked
    runtime (~20 min for an 8 MB file)
- name: OrgMassSpecR
  role: Provides reader/writer functions for spectral libraries tested in the capability
    matrix; supports export to NIST msp
- name: enviGCMS
  role: Reader/writer tested against msp dialects; supports export to the NIST msp
    file format
- name: TargetSearch
  role: Reader/writer tested against msp dialects; supports export to the NIST msp
    file format
- name: MSnbase
  role: Reader function tested against the reference msp snippets and public msp/mgf
    exports (read side only in the comparison)
- name: compMS2Miner
  role: Reader function tested against the reference msp snippets and public msp/mgf
    exports (read side only in the comparison)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9100200
    title: The metaRbolomics Toolbox in Bioconductor and beyond
  source_content_hash: sha256:bd926427a80ee991b8dd8c86458edc4b04b5431cbb311005e2306e642c239039
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spectral-library-format-parsing

## Summary

A method for reading, writing, and round-trip testing spectral libraries in the NIST msp, MoNA msp, and GNPS mgf dialects using R packages, and for assembling a package x dialect/attribute capability matrix. It exists because support for these exchange formats is uneven across R packages (e.g. only metaMS, OrgMassSpecR, enviGCMS, and TargetSearch write NIST msp).

## When to use

Use when R code must parse or export spectral libraries in NIST msp, MoNA msp, or GNPS mgf dialects, or when comparing which R packages can read/write specific spectral-database attributes.

## When NOT to use

- Input is already a parsed in-memory spectral data structure (e.g. an R Spectra/MSnbase object) rather than an msp/mgf file on disk
- The goal is general metabolomics data processing (peak picking, alignment, biostatistics) rather than spectral-library file interchange between R packages
- Working outside R with tools that natively handle all msp/mgf dialects — the capability matrix is specific to the reviewed R packages on CRAN/Bioconductor/GitHub

## Inputs

- Reference spectral-library snippets from Supplemental File S1 (Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME')
- Public MoNA msp export files
- Public GNPS mgf export files
- R packages with spectral-library reader/writer functions: metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner

## Outputs

- Package x dialect/attribute capability matrix as CSV with boolean/enumerated support entries, one row per package
- Per-package parse success/failure records for each msp/mgf dialect
- Round-trip survival records for spectral attributes such as InChIKey and Collision_energy
- Measured export runtimes (e.g. metaMS ~20 min for an 8 MB file)

## How to apply

Parse the reference spectral-library snippets (Listing S1 'basic NIST', Listing S2 'canonical NIST', Listing S3 'RIKEN PRIME' from Supplemental File S1) with the reader functions of each candidate package (metaMS, OrgMassSpecR, enviGCMS, TargetSearch, MSnbase/MSnio, compMS2Miner) and record parse success/failure per package per dialect. Repeat the read benchmark on public MoNA msp and GNPS mgf export files to cover the two public exchange dialects. For each package, run the writer functions on the parsed spectra and record which dialect and which attributes (e.g. InChIKey, Collision_energy) survive import/export round-trips. Time the metaMS export step on the reference content — the review reports ~20 min for an 8 MB file — and assemble the package x dialect/attribute capability matrix as a CSV with boolean/enumerated support entries, one row per package.

## Related tools

- **metaMS** (Reader and writer for NIST msp dialects; its export step is the benchmarked runtime (~20 min for an 8 MB file))
- **OrgMassSpecR** (Provides reader/writer functions for spectral libraries tested in the capability matrix; supports export to NIST msp)
- **enviGCMS** (Reader/writer tested against msp dialects; supports export to the NIST msp file format)
- **TargetSearch** (Reader/writer tested against msp dialects; supports export to the NIST msp file format)
- **MSnbase** (Reader function tested against the reference msp snippets and public msp/mgf exports (read side only in the comparison))
- **compMS2Miner** (Reader function tested against the reference msp snippets and public msp/mgf exports (read side only in the comparison))

## Evaluation signals

- CSV has exactly one row per reviewed package and contains dialect and attribute columns
- All support entries fall in the valid boolean/enumerated range
- metaMS export runtime on the reference content is in the reported order of magnitude (~20 min for an 8 MB file)
- Round-trip test explicitly records whether attributes such as InChIKey and Collision_energy survive import/export
- All three reference dialects (basic NIST, canonical NIST, RIKEN PRIME) plus MoNA msp and GNPS mgf exports were tested per package reader

## Limitations

- metaMS export is slow on real-size files (~20 min for an 8 MB file), which should be budgeted into any workflow using it as a writer
- Only metaMS, OrgMassSpecR, enviGCMS, and TargetSearch among the reviewed packages support export to NIST msp; MSnbase/MSnio and compMS2Miner were evaluated as readers
- Attribute survival differs by dialect: some attributes (e.g. InChIKey, Collision_energy) do not survive import/export round-trips in every package

## Evidence

- [abstract] The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets.: "The programming and statistics environment R has emerged as one of the most popular environments to process and analyse Metabolomics datasets."
- [abstract] This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow: "This review provides an extensive overview of existing packages in R for different steps in a typical computational metabolomics workflow"
- [other] metaMS, OrgMassSpecR, enviGCMS, and TargetSearch are the reviewed R packages that support export to the NIST msp file format, per the review's comparison of spectral-database readers and writers.: [evidence span withheld — anchor status: not_found]
