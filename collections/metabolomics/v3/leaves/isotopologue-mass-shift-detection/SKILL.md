---
name: isotopologue-mass-shift-detection
description: Use when pairwise LC/MS feature tables from an isotope-supplemented condition
  (e.g. 13C6-leucine) and an unlabeled control must be compared to find labeled features
  via a specific mass shift, or to test whether particular lipids derive from a labeled
  precursor.
when_to_use_negative:
- Input is already a fully annotated isotope-enrichment table with shift calls, so
  no pairwise labeled/unlabeled comparison is needed
- The experimental design lacks a matched unlabeled control feature table, since Label
  Finder requires pairwise comparison of supplemented vs. unsupplemented samples
- The goal is general statistical differential-abundance analysis between genotypes
  (e.g. WT vs. hacl-1) rather than isotope-tracing of precursor incorporation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Metaboseek
  role: Interactive analysis platform hosting the LabelFinderModule used in pairwise
    mode to detect isotopologue mass shifts; exports results via Save Table to .csv
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Integrated module used to detect and align molecular features from LC/MS data,
    producing the labeled/unlabeled feature tables consumed by Label Finder
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Used together with xcms (mzR-based) to load xcms results into Metaboseek as
    a Feature Table
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: SIRIUS
  role: Complementary fragment/molecular formula annotation; the paper notes the 'Label
    Finder to identify isotope-labeled compounds' alongside SIRIUS-based formula prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:3726978e9cd930f2d1d68a5ccf2d6ab60f23e45abe5a5ac44685f6133d8d04cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - Metaboseek (LabelFinderModule)
  - SIRIUS
  - xcms
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# isotopologue-mass-shift-detection

## Summary

Detects isotope-labeled metabolites in LC/MS metabolomics by comparing a labeled-supplemented feature table against an unlabeled control with Metaboseek's Label Finder, which flags features showing isotopologue mass shifts (e.g. Δ m/z 5.0167 for 13C5). It traces the origin of lipids such as odd-chain 13:1 and 15:1 species to (or away from) labeled precursor metabolism.

## When to use

Use when pairwise LC/MS feature tables from an isotope-supplemented condition (e.g. 13C6-leucine) and an unlabeled control must be compared to find labeled features via a specific mass shift, or to test whether particular lipids derive from a labeled precursor.

## When NOT to use

- Input is already a fully annotated isotope-enrichment table with shift calls, so no pairwise labeled/unlabeled comparison is needed
- The experimental design lacks a matched unlabeled control feature table, since Label Finder requires pairwise comparison of supplemented vs. unsupplemented samples
- The goal is general statistical differential-abundance analysis between genotypes (e.g. WT vs. hacl-1) rather than isotope-tracing of precursor incorporation

## Inputs

- Centroided LC/MS raw data files in .mzML or .mzXML format
- Preprocessed feature tables with required 'mz' and 'rt' columns, from isotope-supplemented (13C6-Leu) and unlabeled leucine-supplemented sample groups

## Outputs

- Isotope-enrichment table exported as .csv listing features with detected 13C5 mass shifts (Δ m/z 5.0167)
- Label incorporation verdicts for targeted features (e.g. 13:1 and 15:1 monounsaturated lipids)

## How to apply

Load centroided LC/MS raw files (.mzML/.mzXML) into Metaboseek and preprocess them with the integrated xcms module (with MSnbase/mzR support) to produce feature tables that must contain 'mz' and 'rt' columns. Launch the LabelFinderModule in pairwise mode, comparing the 13C6-Leu-supplemented feature table against the unlabeled leucine control table, and filter the output for features whose mass shift matches the 13C5 isotopologue signature (Δ m/z 5.0167). Expected true positives are several hundred 13C5-enriched features including iso-branched fatty acids; features with no detected shift (e.g. hacl-1-enriched 13:1/15:1 monounsaturated lipids) are evidence against derivation from the labeled precursor. Export enrichment results with the Save Table function as a .csv isotope-enrichment table and validate by confirming the expected shift pattern.

## Related tools

- **Metaboseek** (Interactive analysis platform hosting the LabelFinderModule used in pairwise mode to detect isotopologue mass shifts; exports results via Save Table to .csv) — https://github.com/mjhelf/Metaboseek
- **xcms** (Integrated module used to detect and align molecular features from LC/MS data, producing the labeled/unlabeled feature tables consumed by Label Finder) — https://github.com/sneumann/xcms
- **MSnbase** (Used together with xcms (mzR-based) to load xcms results into Metaboseek as a Feature Table) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **SIRIUS** (Complementary fragment/molecular formula annotation; the paper notes the 'Label Finder to identify isotope-labeled compounds' alongside SIRIUS-based formula prediction)

## Examples

```
devtools::install_github("mjhelf/Metaboseek"); then run Metaboseek::Metaboseek() and launch Label Finder in pairwise mode on the 13C6-Leu vs. unlabeled feature tables
```

## Evaluation signals

- The exported enrichment table lists several hundred features carrying the Δ m/z 5.0167 (13C5) mass shift
- Known positive-control features, e.g. iso-branched fatty acids derived from leucine, appear among the 13C5-enriched hits
- The targeted hacl-1-enriched 13:1 and 15:1 monounsaturated lipid rows show no detected label shift, matching the negative finding
- Input feature tables pass schema checks for the required 'mz' and 'rt' columns before Label Finder runs
- The output isotope-enrichment table is exported in .csv format via the Save Table function without errors

## Limitations

- A negative result (no 13C5 label incorporation) shows a lipid does not derive from the labeled precursor's metabolism (e.g. BCFA), but does not identify the actual biosynthetic origin of the compound
- Metabolite annotations in this workflow are limited to confidence Level 3 (tentative structure or compound class assignments only)
- Many detected features remain unidentified, so isotopologue detection alone cannot fully characterize labeled compounds

## Evidence

- [intro] Label Finder supports isotope-tracing workflows: "SIRIUS-based molecular formula and structure prediction, the Label Finder to identify isotope-labeled compounds"
- [readme] xcms and MSnbase load feature tables into Metaboseek: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [methods] xcms detects and aligns molecular features from LC/MS data: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [other] Label Finder trace showed no 13C5 incorporation into odd-chain unsaturated lipids: [evidence span withheld — anchor status: not_found]
