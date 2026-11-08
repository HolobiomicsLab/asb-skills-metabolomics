---
name: metabolite-structure-annotation
description: Use when an LC/MS feature table contains enriched features (e.g. CA-dependent
  metabolites enriched in hacl-1(tm6725)) and you need to collapse CAMERA feature
  groups, predict formulas, and confirm tentative structures from MS2 scans.
when_to_use_negative:
- The workflow aims for definitive Level 1 identification — Metaboseek-based annotation
  here only reaches confidence Level 3 (tentative structure or compound class), so
  a validated structure requirement needs orthogonal confirmation
- Input is already a fully filtered, adduct-free table with no MS2 scans or CAMERA
  group information available, since de-duplication and fragmentation confirmation
  are core to this skill
- Data are not LC/MS features detectable by the integrated xcms module (e.g. direct-infusion
  spectra without retention times)
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: Metaboseek
  role: Interactive platform hosting LoadTableModule, MultiFilterModule, MS2 Browser,
    SaveTableModule, and the Seven Golden Rules formula filters used for structural
    annotation
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Feature detection, alignment, and feature grouping with CAMERA annotation;
    xcms results are loaded into Metaboseek as a Feature Table
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: R package used together with xcms to load xcms results into Metaboseek as
    a Feature Table
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- name: MassTools
  role: R package for mass and molecular formula calculations supporting formula annotation
  repo: https://github.com/mjhelf/MassTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:a2d43aaf94c60c1950e5eb45a3c15c60f3fdef3d92e94d9a9733cd3d363a1df0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - MassTools
  - Metaboseek
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolite-structure-annotation

## Summary

Structurally annotate metabolomics features by combining Metaboseek's statistical filtering, CAMERA-based adduct/isotope de-duplication, SIRIUS molecular formula prediction, and MS2 fragmentation analysis (e.g. neutral-loss confirmation in the MS2 Browser). This is how the HACL-1 study assigned tentative structures (confidence Level 3) to alpha-oxidation-dependent metabolites in C. elegans.

## When to use

Use when an LC/MS feature table contains enriched features (e.g. CA-dependent metabolites enriched in hacl-1(tm6725)) and you need to collapse CAMERA feature groups, predict formulas, and confirm tentative structures from MS2 scans.

## When NOT to use

- The workflow aims for definitive Level 1 identification — Metaboseek-based annotation here only reaches confidence Level 3 (tentative structure or compound class), so a validated structure requirement needs orthogonal confirmation
- Input is already a fully filtered, adduct-free table with no MS2 scans or CAMERA group information available, since de-duplication and fragmentation confirmation are core to this skill
- Data are not LC/MS features detectable by the integrated xcms module (e.g. direct-infusion spectra without retention times)

## Inputs

- LC/MS feature table as .csv with 'mz' and 'rt' columns, or .mskFT with embedded metadata (xcms/MSnbase results loaded into Metaboseek)
- MS2 scans / raw MS-MS data associated with parent features (searchable by ppm and RT tolerance)
- CAMERA-annotated feature groups (adduct and isotope annotations from the integrated XCMS module)

## Outputs

- Filtered feature table in .mskFT format containing the annotated compound set (e.g. the reported 32 CA-derived metabolites) with full processing history
- Tentative structural annotations at confidence Level 3 (structure or compound class assignments) with retention time, m/z, and putative molecular formulae
- MS2 diagnostic evidence such as neutral-loss confirmation for individual features

## How to apply

Load the WT vs hacl-1 CA-supplemented feature table into Metaboseek with LoadTableModule (.csv with 'mz' and 'rt' columns is supported; .mskFT is preferred for embedded metadata). Apply the MultiFilterModule to keep features significantly enriched in hacl-1(tm6725) versus WT using fold-change, intensity, and sample-group filters, then remove adduct and isotope duplicates from CAMERA-annotated feature groups so each compound is represented once. Inspect candidate features (e.g. m/z 185.1183) in the MS2 Browser by locating associated MS2 scans via parent-mass ppm and retention-time tolerance windows, confirming diagnostic fragmentations such as the neutral loss of formic acid for the alpha-hydroxylated citronellic acid derivative. Optionally restrict SIRIUS/FingerID searches to PubChem, and filter predicted formulas with the Seven Golden Rules (element, valence, H/C 0.2–3.1, NOPS/C ratio checks). Save the annotated table with SaveTableModule as .mskFT, which preserves the complete processing history.

## Related tools

- **Metaboseek** (Interactive platform hosting LoadTableModule, MultiFilterModule, MS2 Browser, SaveTableModule, and the Seven Golden Rules formula filters used for structural annotation) — https://github.com/mjhelf/Metaboseek
- **xcms** (Feature detection, alignment, and feature grouping with CAMERA annotation; xcms results are loaded into Metaboseek as a Feature Table) — https://github.com/sneumann/xcms
- **MSnbase** (R package used together with xcms to load xcms results into Metaboseek as a Feature Table) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html
- **MassTools** (R package for mass and molecular formula calculations supporting formula annotation) — https://github.com/mjhelf/MassTools

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- The filtered feature table contains the reported set of 32 CA-derived metabolites, cross-checked against Supplementary Fig. 3
- The m/z 185.1183 feature's MS/MS spectrum shows the characteristic neutral loss of formic acid, confirming the alpha-hydroxylated citronellic acid derivative (7)
- After CAMERA-annotation-based adduct and isotope duplicate removal, each CA-derived compound is represented exactly once in the table
- Predicted molecular formulas pass the Seven Golden Rules heuristics (H/C between 0.2 and 3.1, NOPS/C ratios within bounds, element and valence checks)
- The saved .mskFT file preserves the complete processing history of all filtering and annotation steps

## Limitations

- Metabolite annotations are limited to confidence Level 3 (tentative structure or compound class assignments only)
- Many detected features remain unidentified; the space of yet unidentified biogenic small molecules is large
- Precursor/downstream metabolite relationships (e.g. between beta-branched C11 acids and compounds 11 and 12) remain speculative
- Accession IDs for some deposited MS raw data (refs 43–45) are not provided, complicating direct reanalysis

## Evidence

- [abstract] Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis: "Metaboseek integrates modules for molecular feature detection, statistics, molecular formula prediction, and fragmentation analysis"
- [intro] Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation): "Metaboseek includes an integrated XCMS module for feature detection and feature grouping (with CAMERA annotation)"
- [methods] xcms and MSnbase packages load results as a Feature Table: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [discussion] Level 3 annotation limitation: "facilitates metabolite annotation up to confidence Level 3 for many detected features, enabling tentative structure or compound class assignments"
- [readme] Metaboseek built on xcms with a shiny web GUI: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
