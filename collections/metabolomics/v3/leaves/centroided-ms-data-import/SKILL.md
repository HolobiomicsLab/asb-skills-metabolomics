---
name: centroided-ms-data-import
description: Use when centroided LC/MS raw data files (.mzML or .mzXML) from an untargeted
  HPLC-HRMS/MS experiment must be imported into Metaboseek prior to XCMS feature detection
  and alignment.
when_to_use_negative:
- Input is already a processed xcms feature table (.mskFT) — load it directly into
  Metaboseek instead of re-importing raw data
- Raw data are in profile mode or in vendor formats not covered by .mzML/.mzXML conversion
  — centroid and convert first
- The goal is targeted quantitation or MS/MS-only spectral analysis rather than untargeted
  LC/MS feature detection
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Metaboseek
  role: Provides the graphical Load dialog used to import .mzML/.mzXML files individually
    or from a project folder, and hosts the downstream XCMS module
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: R package whose functions Metaboseek builds on to detect and align molecular
    features from the imported LC/MS data
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used together with xcms to load xcms results into the app as a Feature Table
    after raw data import
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: mzR
  role: Underlying parser through which Metaboseek reads the imported mzML/mzXML raw
    files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:d8a52eec1ad43dc73f8a3fe4855b5796e8fd0242a56b5a01b55e8c70c4b27367
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - mzR
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# centroided-ms-data-import

## Summary

Load centroided LC-HRMS data files (.mzML or .mzXML) into Metaboseek via its Load dialog, individually or from an entire project folder, as the entry point of the untargeted comparative metabolomics workflow that feeds XCMS centWave feature detection.

## When to use

Use when centroided LC/MS raw data files (.mzML or .mzXML) from an untargeted HPLC-HRMS/MS experiment must be imported into Metaboseek prior to XCMS feature detection and alignment.

## When NOT to use

- Input is already a processed xcms feature table (.mskFT) — load it directly into Metaboseek instead of re-importing raw data
- Raw data are in profile mode or in vendor formats not covered by .mzML/.mzXML conversion — centroid and convert first
- The goal is targeted quantitation or MS/MS-only spectral analysis rather than untargeted LC/MS feature detection

## Inputs

- Centroided MS data files in .mzML format
- Centroided MS data files in .mzXML format
- A project folder containing all sample injections (samples plus blank injections)

## Outputs

- Session-loaded raw MS data in Metaboseek ready for XCMS centWave feature detection
- A Metaboseek project folder with run settings and output feature tables (exportable as .mskFT Feature Table)

## How to apply

Use the Metaboseek Load dialog to import centroided MS data files (.mzML or .mzML-converted .mzXML), either individually or by selecting an entire project folder; Metaboseek reads these files through the mzR/MSnbase/xcms stack, so files must already be centroided (profile data must be converted first). For the hacl-1 vs WT C. elegans study this import covered exo- and endo-metabolome injections plus blank injections, which are later needed for blank subtraction. After import, run the XCMS module with centWave feature detection under Metaboseek_default settings to detect and align features across all samples, generating a project folder that stores run settings and output feature tables. Save the session as an .mskFT file so the complete processing history of the xcms run is preserved for reproducibility.

## Related tools

- **Metaboseek** (Provides the graphical Load dialog used to import .mzML/.mzXML files individually or from a project folder, and hosts the downstream XCMS module) — https://github.com/mjhelf/Metaboseek
- **xcms** (R package whose functions Metaboseek builds on to detect and align molecular features from the imported LC/MS data) — https://github.com/sneumann/xcms
- **MSnbase** (Used together with xcms to load xcms results into the app as a Feature Table after raw data import) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **mzR** (Underlying parser through which Metaboseek reads the imported mzML/mzXML raw files)

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); library(Metaboseek)  # then use the app's Load dialog to import .mzML/.mzXML files
```

## Evaluation signals

- Imported files parse without error and their scan-level peak intensities are visible in the Metaboseek browser, confirming the data are centroided and readable via the mzR/MSnbase stack
- The XCMS centWave run over the imported samples yields a feature table on the same order as the study (>700,000 combined molecular features from hacl-1 and WT adult exo- and endo-metabolomes)
- A project folder is generated that stores the run settings and output feature tables
- The session can be saved and reloaded as an .mskFT file with the complete processing history of the xcms run intact
- Blank injections are present in the imported set so the subsequent 10-fold blank subtraction filter can be applied

## Limitations

- Files must already be centroided; Metaboseek import does not itself handle profile-mode or proprietary vendor raw formats
- Feature counts are dataset-specific: the ~150,000 post-filtering figure applies to the full hacl-1/WT study dataset, not to arbitrary subsets
- Metaboseek is under continuous development, so available modules and default settings may change between versions (see the metaboseek.com download/change log)

## Evidence

- [methods] Load centroided MS data files (.mzML or .mzXML) into Metaboseek via the Load dialog, importing either individually or from an entire project folder.: [evidence span withheld — anchor status: not_found]
- [readme] Metaboseek is built on xcms and provides a web-based GUI: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface"
- [methods] xcms results loaded as a Feature Table using xcms and MSnbase: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
