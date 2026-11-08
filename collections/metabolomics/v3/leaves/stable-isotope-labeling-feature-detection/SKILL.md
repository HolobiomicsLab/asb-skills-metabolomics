---
name: stable-isotope-labeling-feature-detection
description: Use when analyzing untargeted LC/MS peak tables from stable isotope-labeling
  experiments (e.g. 12C/13C glucose pairs in XCMS output) to find putatively labeled
  features and annotate them against a metabolite database.
when_to_use_negative:
- The data are not from a stable isotope-labeling experiment with paired unlabeled/labeled
  samples (geoRge relies on comparing isotopic distributions between biologically
  equivalent sample groups)
- Input is raw LC/MS vendor files rather than an XCMS-processed peak set — peak picking,
  alignment and grouping must be done with XCMS first
- You need targeted isotope tracing of known compounds rather than untargeted detection
  of putatively labeled features
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: geoRge
  role: R package providing PuInc_seeker, basepeak_finder and database_query for isotope-labeling
    detection and metabolite annotation
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: upstream peak picking, alignment and grouping producing the XCMSet that geoRge
    consumes
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: runtime language in which geoRge is written and executed
- name: devtools
  role: used to install geoRge from GitHub via install_github
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:066d1644492d7b76e0e661474c1eb231ef8aa71ce4cabeac76e9348f582041a4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - devtools
  - geoRge
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# stable-isotope-labeling-feature-detection

## Summary

geoRge-based R workflow that detects isotopically labeled features in untargeted LC/MS data (XCMS output) by comparing isotopic distributions between unlabeled and labeled biologically equivalent samples, then annotates labeled base peaks with putative metabolites via a database/adduct query.

## When to use

Use when analyzing untargeted LC/MS peak tables from stable isotope-labeling experiments (e.g. 12C/13C glucose pairs in XCMS output) to find putatively labeled features and annotate them against a metabolite database.

## When NOT to use

- The data are not from a stable isotope-labeling experiment with paired unlabeled/labeled samples (geoRge relies on comparing isotopic distributions between biologically equivalent sample groups)
- Input is raw LC/MS vendor files rather than an XCMS-processed peak set — peak picking, alignment and grouping must be done with XCMS first
- You need targeted isotope tracing of known compounds rather than untargeted detection of putatively labeled features

## Inputs

- XCMS feature/peak set (e.g. the mtbls213 XCMSet) from untargeted LC/MS data
- Sample-group tags identifying unlabeled and labeled biologically equivalent samples (ULtag, Ltag)
- Metabolite mass database CSV (e.g. geoRge extdata/ExampleDatabase.csv)
- Adduct table for the ionization mode (e.g. negative-mode adducts: -H, +Cl)

## Outputs

- PuInc_seeker result (s1): putative isotopically labeled feature pairs
- basepeak_finder result (s2): base peaks of labeled features with labeled isotopologue m/z
- database_query hits: feature-to-metabolite mappings including the explaining adduct annotation

## How to apply

Run geoRge in R on an XCMS feature set (peak picking, alignment and grouping done beforehand with xcms). First call PuInc_seeker on the XCMSet with the unlabeled/labeled sample tags (e.g. ULtag 'CELL_Glc12', Ltag 'CELL_Glc13', sep.pos.front=TRUE, fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000) to detect putative isotopically labeled pairs by comparing isotopic distributions of biologically equivalent samples rather than iterating over all MS signals. Then call basepeak_finder on its output with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to identify base peaks of labeled features. Finally, annotate labeled base-peak m/z values with database_query(geoRgeR=s2, adducts=negative, db=db), where db is read from the package's extdata/ExampleDatabase.csv and adducts is the negative-mode adduct table; matches within the ppm.db tolerance are reported with the metabolite and explaining adduct (e.g. -H, +Cl).

## Related tools

- **geoRge** (R package providing PuInc_seeker, basepeak_finder and database_query for isotope-labeling detection and metabolite annotation) — https://github.com/jcapelladesto/geoRge
- **XCMS** (upstream peak picking, alignment and grouping producing the XCMSet that geoRge consumes) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (runtime language in which geoRge is written and executed)
- **devtools** (used to install geoRge from GitHub via install_github)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,UL.atomM=12.0,L.atomM=13.003355,ppm.s=6.5,Basepeak.minInt=2000); hits <- database_query(geoRgeR=s2, adducts=negative, db=db)
```

## Evaluation signals

- Script runs end-to-end in R and each output file (e.g. hits.csv) exists with every annotated hit containing a database metabolite entry and an adduct annotation
- PuInc_seeker applies fc.threshold=1.5 and p.value.threshold=.05 — labeled pairs should pass these fold-change and significance filters
- basepeak_finder tolerance is ppm.s=6.5 and Basepeak.minInt=2000; detected base peaks should respect the 13C mass shift of 13.003355 - 12.0 Da per incorporated atom
- database_query matches must fall within the ppm.db mass tolerance of a database metabolite mass plus a listed negative-mode adduct (e.g. -H, +Cl)
- Results can be cross-checked against X(13)CMS, since both approaches rely entirely on XCMS parameters for feature selection

## Limitations

- geoRge runs on the output structure of the XCMS package, so results depend entirely on the XCMS peak-picking, alignment and grouping parameters chosen
- Annotation via database_query is limited to metabolites present in the supplied database (e.g. ExampleDatabase.csv) and to the adducts in the provided adduct table (negative-mode here)
- Requires biologically equivalent unlabeled and labeled samples; unpaired designs or mixed labeling schemes are outside the demonstrated workflow

## Evidence
