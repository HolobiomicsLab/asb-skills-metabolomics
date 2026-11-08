---
name: stable-isotope-labeling-detection-in-lcms-metabolomics
description: Use when untargeted LC/MS data from a stable isotope-labeling experiment
  (e.g. 12C vs 13C glucose) has been processed with XCMS and you need to enumerate
  features whose isotopic distribution shifts between unlabeled and labeled samples.
when_to_use_negative:
- Data has not been processed through XCMS peak picking/alignment/grouping — geoRge
  runs on the XCMS output structure, not raw mzXML
- The experiment is targeted or lacks biologically equivalent unlabeled/labeled sample
  pairs to compare isotopic distributions
- The goal is de novo isotopologue assignment from theoretical mass differences rather
  than sample-to-sample isotopic distribution comparison
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: R package providing PuInc_seeker, basepeak_finder and database_query to detect
    stable isotope labeling from XCMS feature tables
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: 'Upstream preprocessing: peak picking, alignment and grouping of LC/MS mzXML
    files to produce the XCMSet input for geoRge'
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: devtools
  role: Used to install geoRge from GitHub (install_github('jcapelladesto/geoRge'))
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:8f1938762ec536f268aa45138d08e3fc058b9fa11cd0e2a78936d3eaac15ff2a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - devtools
  - geoRge
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# stable-isotope-labeling-detection-in-lcms-metabolomics

## Summary

geoRge is an R tool for detecting stable isotope labeling in untargeted LC/MS metabolomics by comparing isotopic distributions between biologically equivalent unlabeled and labeled samples in an XCMS feature table, flagging features with new or higher-intensity m/z peaks in labeled samples.

## When to use

Use when untargeted LC/MS data from a stable isotope-labeling experiment (e.g. 12C vs 13C glucose) has been processed with XCMS and you need to enumerate features whose isotopic distribution shifts between unlabeled and labeled samples.

## When NOT to use

- Data has not been processed through XCMS peak picking/alignment/grouping — geoRge runs on the XCMS output structure, not raw mzXML
- The experiment is targeted or lacks biologically equivalent unlabeled/labeled sample pairs to compare isotopic distributions
- The goal is de novo isotopologue assignment from theoretical mass differences rather than sample-to-sample isotopic distribution comparison

## Inputs

- XCMS XCMSet feature table from peak picking, alignment and grouping of mzXML LC/MS files
- Class/sample-group labels identifying unlabeled (e.g. CELL_Glc12) and labeled (e.g. CELL_Glc13) biologically equivalent samples
- Optional metabolite/adduct database CSV (e.g. geoRge extdata/ExampleDatabase.csv) for database_query

## Outputs

- PuInc_seeker result (s1): list of features with putative isotopic incorporation (new or higher-intensity peaks in labeled samples)
- basepeak_finder result (s2): base peaks of the putative isotopically labeled features
- database_query hits: annotated feature identities against an adduct database
- Per-condition counts of isotope-shifted features and their identifiers (e.g. isotope_shift_features_summary.csv)

## How to apply

First process raw mzXML files (e.g. MetaboLights MTBLS213) with XCMS for peak picking, alignment and grouping to obtain the XCMSet feature table. Install geoRge from GitHub (install_github('jcapelladesto/geoRge')) and run PuInc_seeker on the XCMSet with the unlabeled class tag (e.g. ULtag='CELL_Glc12') and labeled tag (Ltag='CELL_Glc13'), using sep.pos.front=TRUE when class names carry a leading prefix, fc.threshold=1.5, p.value.threshold=.05 and PuInc.int.lim=4000 to flag features whose m/z peaks are new or significantly more intense in labeled samples. Then refine with basepeak_finder (PuIncR=s1, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000) to identify the base peaks of the putative isotopically labeled features. Optionally annotate hits with database_query using an adduct database. The rationale is that isotopically enriched compounds change their isotopic distribution relative to unlabeled compounds, which is directly reflected in new m/z peaks and higher-intensity peaks in labeled samples; geoRge compares biologically equivalent samples rather than scanning all MS signals for theoretical mass differences.

## Related tools

- **geoRge** (R package providing PuInc_seeker, basepeak_finder and database_query to detect stable isotope labeling from XCMS feature tables) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream preprocessing: peak picking, alignment and grouping of LC/MS mzXML files to produce the XCMSet input for geoRge) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **devtools** (Used to install geoRge from GitHub (install_github('jcapelladesto/geoRge')))

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,UL.atomM=12.0,L.atomM=13.003355,ppm.s=6.5,Basepeak.minInt=2000)
```

## Evaluation signals

- Output summary table is valid CSV and contains a count row for both glucose conditions (e.g. CELL_Glc12/CELL_Glc13 normal vs high glucose) with nonzero feature counts
- PuInc_seeker only flags features meeting fc.threshold=1.5, p.value.threshold=.05 and PuInc.int.lim=4000
- basepeak_finder candidates match the unlabeled-to-labeled mass shift within ppm.s=6.5 using UL.atomM=12.0 vs L.atomM=13.003355
- Detected isotopically enriched features agree with X(13)CMS outcomes, since both rely entirely on XCMS parameters for feature selection
- Reported per-condition shifted-feature counts are consistent with the published detection of isotopically enriched features by geoRge

## Limitations

- geoRge depends entirely on XCMS parameters for feature selection; feature-detection quality is bounded by upstream XCMS settings
- Class-name parsing depends on the separator/prefix convention (sep.pos.front=TRUE for leading prefixes in class names such as CELL_Glc12/CELL_Glc13)
- Comparison with X(13)CMS was the only independent validation reported; results depend on the chosen thresholds (fc 1.5, p 0.05, PuInc.int.lim 4000)

## Evidence
