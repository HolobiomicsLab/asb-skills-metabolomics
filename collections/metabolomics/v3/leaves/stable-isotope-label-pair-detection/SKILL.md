---
name: stable-isotope-label-pair-detection
description: Use when you have an XCMS-processed XCMSet with paired unlabeled/labeled
  sample groups (e.g., 12C vs 13C glucose) and need to find feature pairs showing
  shifted m/z and enrichment-consistent intensity changes.
when_to_use_negative:
- Input is raw mzXML without prior XCMS peak picking, alignment and grouping — geoRge
  runs on the XCMS output structure, not raw spectra.
- No unlabeled biologically equivalent control samples exist to compare isotopic distributions
  against, since geoRge's contrast requires UL/L sample pairs.
- Labeling is with an element other than the configured atom masses (e.g., not 12C/13C
  at 12.0 vs 13.003355 Da) unless parameters are retuned accordingly.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: geoRge
  role: Provides PuInc_seeker, basepeak_finder and database_query functions that detect
    putative isotope-labeled pairs from the XCMSet
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Performs peak picking, alignment and grouping upstream so that the XCMSet
    feature table feeds PuInc_seeker
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Language in which geoRge is written and run; installed via devtools install_github
  repo: https://github.com/jcapelladesto/geoRge
- name: devtools
  role: Used to install geoRge from GitHub (install_github("jcapelladesto/geoRge"))
  repo: https://github.com/jcapelladesto/geoRge
- name: X(13)CMS
  role: Alternative tool run on the same XCMS feature table for cross-tool comparison
    of detected labeled features
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:8f587a35024615acb6f1bed301ac134f6b8934933f4c9b55e7d17d269c9c0a9d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - MSnbase
  - R
  - X(13)CMS
  - XCMS
  - devtools
  - geoRge
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.92
n_claims: 25
---

# stable-isotope-label-pair-detection

## Summary

Detects putative isotopically-labeled feature pairs in untargeted LC/MS metabolomics by comparing isotopic distributions between unlabeled and labeled biologically equivalent samples on an XCMS feature table (geoRge, Analytical Chemistry 2015).

## When to use

Use when you have an XCMS-processed XCMSet with paired unlabeled/labeled sample groups (e.g., 12C vs 13C glucose) and need to find feature pairs showing shifted m/z and enrichment-consistent intensity changes.

## When NOT to use

- Input is raw mzXML without prior XCMS peak picking, alignment and grouping — geoRge runs on the XCMS output structure, not raw spectra.
- No unlabeled biologically equivalent control samples exist to compare isotopic distributions against, since geoRge's contrast requires UL/L sample pairs.
- Labeling is with an element other than the configured atom masses (e.g., not 12C/13C at 12.0 vs 13.003355 Da) unless parameters are retuned accordingly.

## Inputs

- XCMS XCMSet object (peak picking, alignment and grouping already applied) built from mzXML files such as MTBLS213
- Sample-group tags identifying unlabeled (ULtag, e.g. CELL_Glc12) and labeled (Ltag, e.g. CELL_Glc13) biologically equivalent samples
- Atom masses for the label element (UL.atomM=12.0, L.atomM=13.003355)

## Outputs

- PuIncR object (s1) listing putative isotopically-labeled feature pairs with fold-change and p-value evidence
- basepeak_finder result (s2) identifying base peaks of labeled features at 6.5 ppm tolerance
- Optional annotated hits from database_query against negative-mode adducts and ExampleDatabase.csv

## How to apply

Start from an XCMS peak-picked, aligned and grouped XCMSet (e.g., the MTBLS213 mzXML files) where unlabeled and labeled biologically equivalent samples are tagged (ULtag="CELL_Glc12", Ltag="CELL_Glc13"). Run geoRge's PuInc_seeker with sep.pos.front=TRUE to handle group-name prefixes, an fold-change threshold (fc.threshold=1.5), significance threshold (p.value.threshold=.05), and a minimum intensity limit (PuInc.int.lim=4000) to flag features that gain intensity and shift by ~1.003355 Da per label atom in labeled samples. Feed the resulting PuIncR object into basepeak_finder with UL.atomM=12.0, L.atomM=13.003355 and ppm.s=6.5 to locate the base peaks of labeled features, then optionally database_query against a negative-mode adduct list and ExampleDatabase.csv for annotation. The rationale is to use the labeled-vs-unlabeled isotopic-distribution contrast rather than iterating over all MS signals with theoretical mass differences.

## Related tools

- **geoRge** (Provides PuInc_seeker, basepeak_finder and database_query functions that detect putative isotope-labeled pairs from the XCMSet) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Performs peak picking, alignment and grouping upstream so that the XCMSet feature table feeds PuInc_seeker) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Language in which geoRge is written and run; installed via devtools install_github) — https://github.com/jcapelladesto/geoRge
- **devtools** (Used to install geoRge from GitHub (install_github("jcapelladesto/geoRge"))) — https://github.com/jcapelladesto/geoRge
- **X(13)CMS** (Alternative tool run on the same XCMS feature table for cross-tool comparison of detected labeled features)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000)
```

## Evaluation signals

- PuInc_seeker runs without error on the XCMSet and returns a PuIncR object containing labeled/unlabeled feature pairs at fc.threshold=1.5 and p.value.threshold=.05.
- basepeak_finder output contains base peaks whose m/z shifts match 13.003355 - 12.0 = 1.003355 Da per label atom within ppm.s=6.5, with Basepeak.minInt=2000 applied.
- Detected pairs are reproducible on the package example dataset mtbls213 with the documented ULtag/Ltag group names.
- Agreement/discrepancy with X(13)CMS on the same XCMS feature table is tabulated, since both approaches rely entirely on XCMS parameters for feature selection.
- database_query hits are restricted to the negative-mode adduct list and entries in ExampleDatabase.csv.

## Limitations

- geoRge depends entirely on XCMS parameters (ppm, peakwidth, mzwid, minfrac, bw) for feature selection; mis-tuned XCMS parameters propagate into label detection.
- The authors note XCMS parameters were 'tuned by trial and error', so results are sensitive to manual parameter choice.
- Requires correctly spelled ULtag/Ltag group names matching the XCMSet phenoData prefixes (sep.pos.front=TRUE), e.g. CELL_Glc12/CELL_Glc13; mismatched tags will fail to form pairs.
- geoRge 1.0 changed function arguments; older scripts using the original argument names may break — check help(function) for current usage.

## Evidence
