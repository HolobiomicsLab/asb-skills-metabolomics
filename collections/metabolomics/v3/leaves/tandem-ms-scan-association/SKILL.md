---
name: tandem-ms-scan-association
description: Use when you have an xcms-derived feature table (csv with 'mz' and 'rt'
  columns) plus centroided LC/MS raw data (.mzML) and need to link MS2 spectra to
  features before MS/MS-based annotation such as Find Patterns or spectral similarity
  searches.
when_to_use_negative:
- When only MS1 data without data-dependent MS2 scans are available — no spectra can
  be associated to features
- When the feature table already contains MS2 spectra association from a prior Metaboseek
  session saved as .mskFT and no raw data changed
- When the goal is MS1-only statistics or feature detection, which is handled by the
  xcms module directly
edam_operation: http://edamontology.org/operation_3645
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Platform providing the MS2 Browser module that associates MS2 scans with feature-table
    entries using customizable m/z and retention-time tolerances
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Produces the feature table (mz, rt entries) from LC/MS data that MS2 scans
    are associated to
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used to load xcms results into Metaboseek as a Feature Table
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:ef0b53df911324eab425d267b89880f089c2e523d5759450e3885b595278399a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - FindPatternsModule
  - MSnbase
  - Metaboseek
  - xcms
  edam_operation: http://edamontology.org/operation_3645
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tandem-ms-scan-association

## Summary

Associates MS2 scans with feature-table entries in Metaboseek so that fragment-based pattern searches (e.g., the GLEA product ion at m/z 333.0592) can be run over the MS/MS spectra belonging to each LC/MS molecular feature.

## When to use

Use when you have an xcms-derived feature table (csv with 'mz' and 'rt' columns) plus centroided LC/MS raw data (.mzML) and need to link MS2 spectra to features before MS/MS-based annotation such as Find Patterns or spectral similarity searches.

## When NOT to use

- When only MS1 data without data-dependent MS2 scans are available — no spectra can be associated to features
- When the feature table already contains MS2 spectra association from a prior Metaboseek session saved as .mskFT and no raw data changed
- When the goal is MS1-only statistics or feature detection, which is handled by the xcms module directly

## Inputs

- Centroided LC/MS data files (.mzML or other Metaboseek-supported formats)
- Feature table (csv with 'mz' and 'rt' columns) from xcms feature detection
- MS2 scans embedded in the raw LC/MS files (data-dependent acquisition)

## Outputs

- Feature table with MS2 scans associated to each entry (via the MS2 Browser)
- Annotated feature table (.mskFT with processing history and metadata, plus exported .csv)
- Pattern-match hit columns (e.g., Find Patterns hits) in the feature table

## How to apply

Load centroided LC/MS files (.mzML or supported formats) and the feature table into Metaboseek via the Load module, then use the MS2 Browser to associate MS2 scans with feature-table entries using the default tolerances: 5 ppm parent-ion m/z tolerance and a 200 s retention time window (both customizable). This step attaches the underlying MS/MS spectra to each feature row so downstream modules like FindPatternsModule can search the GLEA-characteristic product ion at m/z 333.0592 together with phosphoglycerol- and glucose-derived fragments across SN2/SN3/SN4 subnetwork features. The rationale is that feature detection with xcms alone yields only mz/rt rows without spectra; the association bridges MS1 features to MS2 evidence, enabling pattern-match hits to be written back into new interactive columns of the feature table. After association, annotate diagnostic ions (e.g., beta-hydroxylation products at m/z 376.1016 and 418.1123) among matched spectra before saving the annotated table as .mskFT and exporting .csv.

## Related tools

- **Metaboseek** (Platform providing the MS2 Browser module that associates MS2 scans with feature-table entries using customizable m/z and retention-time tolerances) — https://github.com/mjhelf/Metaboseek
- **xcms** (Produces the feature table (mz, rt entries) from LC/MS data that MS2 scans are associated to) — https://github.com/sneumann/xcms
- **MSnbase** (Used to load xcms results into Metaboseek as a Feature Table) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); msk <- MskDir(); # Load .mzML + feature table csv, then use MS2 Browser: 5 ppm m/z tolerance, 200 s RT window
```

## Evaluation signals

- Each feature-table row of interest has at least one associated MS2 scan within 5 ppm parent m/z and the 200 s RT window
- Downstream Find Patterns runs return non-zero hits for features whose spectra contain m/z 333.0592 plus phosphoglycerol- and glucose-derived fragments
- Diagnostic ions m/z 376.1016 and m/z 418.1123 (SN3 beta-hydroxylation products) appear among matched spectra
- Saved .mskFT retains processing history and metadata, and exported .csv contains the new hit columns
- More than 100 GLEA-like molecular features are identified across SN2, SN3, and SN4 after association

## Limitations

- Association tolerances (5 ppm, 200 s) are customizable defaults — poorly tuned values can miss or falsely associate MS2 scans
- Only MS2 spectra present in the acquired raw data can be associated; features lacking data-dependent MS2 coverage get no spectra
- Metaboseek metabolite annotation is limited to confidence Level 3 (tentative structure or compound class assignments)

## Evidence

- [methods] Associate MS2 scans with feature-table entries using the MS2 Browser default tolerances (5 ppm parent-ion m/z tolerance, 200 s retention time window, both customizable): [evidence span withheld — anchor status: not_found]
- [methods] You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [results] The Find Patterns function enabled rapid identification of more than 100 GLEA-like molecular features distributed across SN2 and two additional subnetworks, SN3 and SN4: [evidence span withheld — anchor status: not_found]
- [readme] Metaboseek is an interactive tool to analyze and browse your mass spectrometry data. It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface: "Metaboseek is an interactive tool to analyze and browse your mass spectrometry data."
- [discussion] facilitates metabolite annotation up to confidence Level 3 for many detected features, enabling tentative structure or compound class assignments: "facilitates metabolite annotation up to confidence Level 3 for many detected features, enabling tentative structure or compound class assignments"
