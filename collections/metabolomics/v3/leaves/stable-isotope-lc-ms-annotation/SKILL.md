---
name: stable-isotope-lc-ms-annotation
description: Use when you have XCMS-processed untargeted LC/MS data from paired unlabeled/labeled
  tracer experiments (e.g., 12C vs 13C glucose) and need to detect isotopically enriched
  features and annotate their base peaks.
when_to_use_negative:
- Your data are not XCMS feature objects — geoRge runs on the output structure of
  the XCMS package, so raw spectra or non-XCMS peak tables are out of scope
- You lack biologically equivalent unlabeled and labeled sample groups — geoRge compares
  isotopic distributions between such pairs and cannot infer labeling from labeled
  samples alone
- You need targeted isotope-tracer quantification rather than untargeted detection
  of putative labeling in unannotated features
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: geoRge
  role: Core R package providing PuInc_seeker, basepeak_finder, refinegeoRge and database_query
    for untargeted isotope labeling detection and annotation
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment and grouping of raw LC/MS mzXML files into
    the feature set geoRge consumes
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: devtools
  role: Used to install geoRge from GitHub via install_github('jcapelladesto/geoRge')
  repo: https://github.com/jcapelladesto/geoRge
- name: X(13)CMS
  role: Comparator method used to benchmark geoRge's labeling detections since both
    rely entirely on XCMS parameters for feature selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:7ae30fa4b96d5f0457b3c18dfc8aa6965766658309ff361c15a3e5d7bbcf740b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - X(13)CMS
  - XCMS
  - devtools
  - geoRge
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# stable-isotope-lc-ms-annotation

## Summary

Detects and annotates stable isotope labeling in LC/MS-based untargeted metabolomics using geoRge, an R tool that compares isotopic distributions between unlabeled and labeled biologically equivalent samples on XCMS output, rather than scanning all MS signals for theoretical mass differences.

## When to use

Use when you have XCMS-processed untargeted LC/MS data from paired unlabeled/labeled tracer experiments (e.g., 12C vs 13C glucose) and need to detect isotopically enriched features and annotate their base peaks.

## When NOT to use

- Your data are not XCMS feature objects — geoRge runs on the output structure of the XCMS package, so raw spectra or non-XCMS peak tables are out of scope
- You lack biologically equivalent unlabeled and labeled sample groups — geoRge compares isotopic distributions between such pairs and cannot infer labeling from labeled samples alone
- You need targeted isotope-tracer quantification rather than untargeted detection of putative labeling in unannotated features

## Inputs

- XCMS feature set (xcmsSet object) derived from raw LC/MS mzXML files (e.g., MetaboLights MTBLS213)
- Sample metadata tags identifying unlabeled and labeled biologically equivalent groups (e.g., ULtag='CELL_Glc12', Ltag='CELL_Glc13')
- geoRge PuInc result object (output of PuInc_seeker feeding basepeak_finder)
- Optional compound database CSV with expected adducts (e.g., package extdata/ExampleDatabase.csv)

## Outputs

- PuInc result object listing putative isotopically labelled feature pairs with fold-change and p-value statistics
- Base-peak assignments for labelled features (UL.atomM 12.0 vs L.atomM 13.003355 within ppm.s 6.5)
- Refined feature table (e.g., refined_george_results.csv) with accepted labeled-atom counts and isotope-correlation values per feature
- Database annotation hits mapping base peaks to compounds via adduct rules

## How to apply

First process raw mzXML files (e.g., MetaboLights MTBLS213) with XCMS for peak picking, alignment and grouping, then run PuInc_seeker on the XCMS set with ULtag/Ltag sample tags (e.g., CELL_Glc12 vs CELL_Glc13), fc.threshold=1.5, p.value.threshold=0.05 and PuInc.int.lim=4000 to detect putative isotopically labelled pairs. Next call basepeak_finder with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to identify the base peaks of labelled features. Optionally refine assignments with refinegeoRge using minAtoms, maxAtoms and minCorr thresholds to retain only base-peak assignments passing labeled-atom count and isotope-correlation criteria, and optionally query a compound database via database_query (e.g., negative-mode adducts against ExampleDatabase.csv). Rationale: geoRge leverages the isotopic distribution shift between unlabeled and labeled equivalents — new m/z peaks and higher-intensity peaks in labeled samples — so correct sample tagging is the critical decision point. Export refined results (accepted labeled-atom counts and correlation values per feature) to CSV for downstream reporting.

## Related tools

- **geoRge** (Core R package providing PuInc_seeker, basepeak_finder, refinegeoRge and database_query for untargeted isotope labeling detection and annotation) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment and grouping of raw LC/MS mzXML files into the feature set geoRge consumes) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **devtools** (Used to install geoRge from GitHub via install_github('jcapelladesto/geoRge')) — https://github.com/jcapelladesto/geoRge
- **X(13)CMS** (Comparator method used to benchmark geoRge's labeling detections since both rely entirely on XCMS parameters for feature selection)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,UL.atomM=12.0,L.atomM=13.003355,ppm.s=6.5,Basepeak.minInt=2000)
```

## Evaluation signals

- Putative labeled pairs pass PuInc_seeker thresholds: fold-change ≥ fc.threshold 1.5 and p-value ≤ 0.05 with intensity above PuInc.int.lim 4000
- Base-peak candidates satisfy the mass search: 12.0 vs 13.003355 neutron mass difference matched within ppm.s = 6.5 and base-peak intensity above Basepeak.minInt 2000
- refinegeoRge output retains only base-peak assignments passing minAtoms, maxAtoms and minCorr labeled-atom-count and isotope-correlation criteria
- Exported refined_george_results.csv contains per-feature labeled-atom counts and correlation values for every accepted base peak
- Detected labeled features are broadly concordant with X(13)CMS outcomes on the same XCMS feature set

## Limitations

- geoRge relies entirely on XCMS parameters for feature selection, so upstream peak picking, alignment and grouping settings directly bound its detections
- Requires unlabeled and labeled biologically equivalent samples for isotopic distribution comparison; labeling signals are judged by new m/z peaks and higher-intensity peaks in labeled samples rather than theoretical mass scans
- The README notes geoRge 1.0 changed function arguments ('same functions, less arguments'), so older parameter lists may need checking against the updated documentation via help(function)

## Evidence
