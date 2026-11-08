---
name: lcms-isotope-label-detection
description: Use when you have XCMS-processed untargeted LC/MS feature data from paired
  unlabeled vs. labeled samples (e.g. CELL_Glc12 vs CELL_Glc13) and need to flag isotopically
  enriched features and their base-peak isotope assignments.
when_to_use_negative:
- Input is raw vendor LC/MS data rather than an XCMS-processed feature/peak table
  — run XCMS peak picking, alignment and grouping first.
- You need isotope tracing of elements whose mass shift differs from 13C (L.atomM
  must be set to the labeled atom's exact mass; the example workflow is parameterized
  for 12C/13C).
- There are no biologically equivalent unlabeled control samples to compare against
  — geoRge's core logic compares isotopic distributions between unlabeled and labeled
  samples.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: geoRge
  role: R package providing PuInc_seeker, basepeak_finder and database_query for isotope-label
    detection and base-peak assignment
  repo: github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment and grouping of LC/MS mzXML data to produce
    the feature set geoRge operates on
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Execution language; geoRge is written in R and installed via devtools install_github
  repo: github.com/jcapelladesto/geoRge
- name: devtools
  role: Used to install geoRge from GitHub (install.packages('devtools'); install_github('jcapelladesto/geoRge'))
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:9d6b51a1adf2a457933783c766a5f83bd673ecb4ac60a2a5a2f7c0e66de46336
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - XCMS
  - devtools
  - geoRge
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lcms-isotope-label-detection

## Summary

Detect stable isotope labeling in untargeted LC/MS metabolomics by comparing isotopic distributions between unlabeled and labeled biologically equivalent samples using geoRge on XCMS feature output, then assign monoisotopic base peaks to putatively labeled features (e.g. 13C glucose-labeled ARPE-19 cells, MTBLS213).

## When to use

Use when you have XCMS-processed untargeted LC/MS feature data from paired unlabeled vs. labeled samples (e.g. CELL_Glc12 vs CELL_Glc13) and need to flag isotopically enriched features and their base-peak isotope assignments.

## When NOT to use

- Input is raw vendor LC/MS data rather than an XCMS-processed feature/peak table — run XCMS peak picking, alignment and grouping first.
- You need isotope tracing of elements whose mass shift differs from 13C (L.atomM must be set to the labeled atom's exact mass; the example workflow is parameterized for 12C/13C).
- There are no biologically equivalent unlabeled control samples to compare against — geoRge's core logic compares isotopic distributions between unlabeled and labeled samples.

## Inputs

- XCMS feature set object (e.g. the mtbls213 data shipped with geoRge, or built from MTBLS213 mzXML files via XCMS peak picking, alignment and grouping)
- Sample-class tags identifying unlabeled vs. labeled groups (e.g. ULtag='CELL_Glc12', Ltag='CELL_Glc13')
- PuInc_seeker result object (s1) as input to basepeak_finder
- Optional metabolite database CSV (e.g. geoRge extdata/ExampleDatabase.csv) and adduct list (negative mode) for database_query

## Outputs

- PuInc_seeker result (s1): table of putative isotopically labeled feature pairs with fold-change and p-value filtering
- basepeak_finder result (s2): feature-to-base-peak m/z mapping with labeled-vs-unlabeled intensity ratios
- database_query hits table when a metabolite database is supplied
- Exported CSV, e.g. george_mtbls213_isotope_table.csv or basepeak_assignments.csv

## How to apply

Start from an XCMS feature set (peak picking, alignment and grouping) such as the MTBLS213 ARPE-19 data, then run geoRge's PuInc_seeker with the sample-name tags of the unlabeled and labeled classes (ULtag='CELL_Glc12', Ltag='CELL_Glc13', sep.pos.front=TRUE, fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000) to detect putative isotopically labeled feature pairs. Because isotopically enriched compounds show new m/z peaks and higher intensity peaks in labeled samples, geoRge compares isotopic distributions between the two biologically equivalent sample groups rather than iterating all MS signals with theoretical mass differences. Feed the PuInc_seeker result into basepeak_finder with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to match each putatively labeled feature to its monoisotopic base peak. Optionally annotate hits against a metabolite database (ExampleDatabase.csv with negative-mode adducts) via database_query. Export the resulting table of labeled features with base-peak assignments and labeled-vs-unlabeled intensity ratios as a CSV.

## Related tools

- **geoRge** (R package providing PuInc_seeker, basepeak_finder and database_query for isotope-label detection and base-peak assignment) — github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment and grouping of LC/MS mzXML data to produce the feature set geoRge operates on) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Execution language; geoRge is written in R and installed via devtools install_github) — github.com/jcapelladesto/geoRge
- **devtools** (Used to install geoRge from GitHub (install.packages('devtools'); install_github('jcapelladesto/geoRge')))

## Examples

```
library(geoRge); s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1, XCMSet=mtbls213, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000)
```

## Evaluation signals

- The exported CSV contains labeled features each assigned a monoisotopic base-peak m/z; verify the output exists and is well-formed.
- Assignment parameters recorded match the spec: UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000.
- PuInc_seeker filters applied: fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000; features failing these thresholds should not appear in s1.
- Base-peak candidates lie within 6.5 ppm of the isotope-shifted m/z consistent with a 13C mass difference (13.003355 − 12.0).
- Cross-check agreement with an independent method such as X(13)CMS, as the paper compares geoRge with X(13)CMS since both rely entirely on XCMS parameters for feature selection.

## Limitations

- geoRge runs on the output structure of XCMS, so results depend entirely on XCMS peak picking/alignment/grouping parameters for feature selection.
- Requires matched unlabeled and labeled biologically equivalent samples; detection relies on changes in isotopic distribution (new m/z peaks, higher intensity peaks in labeled samples).
- Base-peak assignment is bounded by the 6.5 ppm tolerance and Basepeak.minInt=2000 intensity floor; features below these limits are not assigned.
- The README's example invocation uses slightly different parameter names (sep.pos.front=TRUE) than the article's sep.pos='f' — check help(function) for the geoRge 1.0 argument names.

## Evidence
