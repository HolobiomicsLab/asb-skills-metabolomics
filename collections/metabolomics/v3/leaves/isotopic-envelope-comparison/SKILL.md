---
name: isotopic-envelope-comparison
description: Use when untargeted LC/MS feature tables from pairs of biologically equivalent
  samples (one grown on a 12C substrate, one on a 13C-labeled substrate) must be screened
  for isotope-label-driven shifts in isotopic envelope.
when_to_use_negative:
- Input is already an annotated isotope table or the data come from targeted MS/MS
  labeling experiments rather than untargeted XCMS output
- Samples are not biologically equivalent unlabeled/labeled pairs — geoRge requires
  both conditions to compare isotopic distributions
- Feature list was produced by a tool other than XCMS peak picking/alignment/grouping,
  since geoRge relies entirely on XCMS parameters for feature selection
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: geoRge
  role: R package providing PuInc_seeker, basepeak_finder and database_query for isotope-label
    detection on XCMS output
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: performs peak picking, alignment and grouping on raw mzXML files to produce
    the XCMS set consumed by geoRge
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: runtime language in which geoRge is written and executed
- name: devtools
  role: used to install geoRge from GitHub via install_github
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:0171a2712c81b42fb18ce493480b29900dde89707feb8e4e3c6860557d3cc37c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - devtools
  - geoRge
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# isotopic-envelope-comparison

## Summary

Detect stable isotope labeling in untargeted LC/MS metabolomics by comparing isotopic distributions between unlabeled and labeled biologically equivalent samples, using geoRge's PuInc_seeker and basepeak_finder functions on an XCMS feature set (e.g. MTBLS213 ARPE-19 glucose labeling data).

## When to use

Use when untargeted LC/MS feature tables from pairs of biologically equivalent samples (one grown on a 12C substrate, one on a 13C-labeled substrate) must be screened for isotope-label-driven shifts in isotopic envelope.

## When NOT to use

- Input is already an annotated isotope table or the data come from targeted MS/MS labeling experiments rather than untargeted XCMS output
- Samples are not biologically equivalent unlabeled/labeled pairs — geoRge requires both conditions to compare isotopic distributions
- Feature list was produced by a tool other than XCMS peak picking/alignment/grouping, since geoRge relies entirely on XCMS parameters for feature selection

## Inputs

- XCMS xcmsSet object (e.g. mtbls213) built from mzXML files of paired unlabeled/labeled samples
- Sample-group tags identifying the unlabeled (ULtag='CELL_Glc12') and labeled (Ltag='CELL_Glc13') conditions
- Optional metabolite database CSV (e.g. geoRge extdata/ExampleDatabase.csv) with adduct definitions

## Outputs

- PuInc_seeker result (s1): table of putative isotopically labeled feature pairs with fold-change and p-value
- basepeak_finder result (s2): table assigning base peaks to labeled features using UL.atomM/L.atomM mass difference and ppm.s tolerance
- Optional database_query hit table mapping labeled features to metabolites/adducts

## How to apply

First process raw mzXML files with XCMS for peak picking, alignment and grouping (or load the pre-built mtbls213 XCMSSet shipped with geoRge). Run PuInc_seeker on the XCMS set with ULtag='CELL_Glc12' and Ltag='CELL_Glc13', sep.pos.front=TRUE, fc.threshold=1.5, p.value.threshold=.05 and PuInc.int.lim=4000 to find feature pairs whose intensity changes between unlabeled and labeled samples in a pattern consistent with 13C enrichment. Then run basepeak_finder on the PuInc_seeker result with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to assign the monoisotopic base peak to each labeled feature. The rationale is that geoRge compares isotopic distributions across unlabeled vs labeled biologically equivalent samples instead of scanning all MS signals for theoretical mass differences; optionally annotate hits with database_query against a metabolite/adduct database (ExampleDatabase.csv, negative-mode adducts).

## Related tools

- **geoRge** (R package providing PuInc_seeker, basepeak_finder and database_query for isotope-label detection on XCMS output) — https://github.com/jcapelladesto/geoRge
- **XCMS** (performs peak picking, alignment and grouping on raw mzXML files to produce the XCMS set consumed by geoRge) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (runtime language in which geoRge is written and executed)
- **devtools** (used to install geoRge from GitHub via install_github)

## Examples

```
install.packages("devtools",dependencies=TRUE); library(devtools); install_github("jcapelladesto/geoRge"); library(geoRge); s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,UL.atomM=12.0,L.atomM=13.003355,ppm.s=6.5,Basepeak.minInt=2000)
```

## Evaluation signals

- PuInc_seeker returns a non-empty result (s1) on the packaged mtbls213 example with ULtag='CELL_Glc12', Ltag='CELL_Glc13'
- basepeak_finder output contains mass-difference assignments consistent with UL.atomM=12.0 vs L.atomM=13.003355 within ppm.s=6.5
- Labeled-vs-unlabeled intensity fold changes exceed fc.threshold=1.5 with p.value.threshold=0.05 for reported pairs
- Base-peak intensities clear Basepeak.minInt=2000 and PuInc features clear PuInc.int.lim=4000
- Optional cross-check: compare geoRge feature list against X(13)CMS results, since both rely entirely on XCMS parameters for feature selection

## Limitations

- geoRge requires unlabeled and labeled biologically equivalent samples; it cannot detect labeling without a paired unlabeled control
- Detection quality depends entirely on the XCMS peak picking, alignment and grouping parameters used upstream
- README notes label_compare() had to be patched in geoRge 1.0 — verify installed version; issues should be raised at the GitHub issue tracker
- Tracer-specific: default example assumes 13C glucose labeling (13.003355 vs 12.0 mass difference); other isotopes need adjusted L.atomM/UL.atomM

## Evidence
