---
name: isotopic-distribution-shift-analysis
description: Use when an XCMS peak table from an untargeted LC/MS stable-isotope experiment
  must be scanned for features whose m/z peaks are new or significantly more intense
  in labeled vs unlabeled samples (e.g. 13C-glucose vs 12C-glucose classes).
when_to_use_negative:
- Input is already an annotated/identified compound list rather than an XCMS feature
  table — geoRge runs on the XCMS output structure of raw-feature peak picking, alignment
  and grouping
- The experiment has no biologically equivalent unlabeled/labeled sample pair — geoRge
  requires unlabeled and labeled equivalent samples to compare isotopic distributions,
  unlike theoretical-mass-difference approaches
- The goal is targeted isotope-tracer quantification with predefined metabolites rather
  than untargeted detection of labeling presence
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: geoRge
  role: Core R package; PuInc_seeker flags putative isotopically labeled features
    and basepeak_finder identifies their base peaks
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment and grouping that produces the XCMSet feature
    table consumed by geoRge
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Runtime language; geoRge is installed from GitHub via devtools and runs on
    XCMS output
  repo: https://github.com/jcapelladesto/geoRge
- name: X(13)CMS
  role: Comparator tool used to benchmark geoRge's shifted-feature detection since
    both rely entirely on XCMS parameters for feature selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:83664c8605bb5b0573e90900f37a4a9f2f4d46baa9d0eaa2ffedcfd767312405
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - X(13)CMS
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# isotopic-distribution-shift-analysis

## Summary

Detects stable-isotope (13C) labeling in untargeted LC/MS metabolomics by comparing isotopic distributions between biologically equivalent unlabeled and labeled samples using geoRge's PuInc_seeker and basepeak_finder on XCMS feature tables, e.g. the MTBLS213 ARPE-19 dataset grown on 12C vs 13C glucose.

## When to use

Use when an XCMS peak table from an untargeted LC/MS stable-isotope experiment must be scanned for features whose m/z peaks are new or significantly more intense in labeled vs unlabeled samples (e.g. 13C-glucose vs 12C-glucose classes).

## When NOT to use

- Input is already an annotated/identified compound list rather than an XCMS feature table — geoRge runs on the XCMS output structure of raw-feature peak picking, alignment and grouping
- The experiment has no biologically equivalent unlabeled/labeled sample pair — geoRge requires unlabeled and labeled equivalent samples to compare isotopic distributions, unlike theoretical-mass-difference approaches
- The goal is targeted isotope-tracer quantification with predefined metabolites rather than untargeted detection of labeling presence

## Inputs

- Raw LC/MS mzXML files deposited in MetaboLights (MTBLS213)
- XCMS XCMSet feature table from peak picking, alignment and grouping
- Class/tag names identifying unlabeled and labeled biologically equivalent samples (e.g. CELL_Glc12, CELL_Glc13) with the leading class-name prefix separator (sep.pos='f')

## Outputs

- PuInc_seeker result: features with putative isotopic incorporation (new or significantly more intense m/z peaks in labeled samples)
- basepeak_finder result: base peaks of putative isotopically labeled features (ppm.s=6.5, Basepeak.minInt=2000)
- Per-condition summary table (isotope_shift_features_summary.csv) with shifted-feature counts and identifiers

## How to apply

Process raw mzXML files (e.g. MetaboLights MTBLS213) with XCMS for peak picking, alignment and grouping to obtain the XCMSet feature table. Run geoRge::PuInc_seeker on the XCMSet with ULtag='CELL_Glc12' and Ltag='CELL_Glc13', sep.pos='f' (leading class-name prefix), fc.threshold=1.5, p.value.threshold=0.05 and PuInc.int.lim=4000 to flag features whose isotopic distribution shifts — new or higher-intensity m/z peaks in the labeled condition. Then run basepeak_finder with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to identify the base peaks of the putative labeled features and refine the list. Optionally annotate hits via database_query against a metabolite database (e.g. ExampleDatabase.csv with negative-mode adducts). Aggregate the putative-incorporation and basepeak features to count shifted features per condition (e.g. normal vs high glucose) and write per-condition counts and feature identifiers to a summary table such as isotope_shift_features_summary.csv. The rationale is that isotopically enriched compounds change their isotopic distribution relative to unlabeled compounds, directly reflected as new m/z peaks and higher-intensity peaks in labeled samples.

## Related tools

- **geoRge** (Core R package; PuInc_seeker flags putative isotopically labeled features and basepeak_finder identifies their base peaks) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment and grouping that produces the XCMSet feature table consumed by geoRge) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Runtime language; geoRge is installed from GitHub via devtools and runs on XCMS output) — https://github.com/jcapelladesto/geoRge
- **X(13)CMS** (Comparator tool used to benchmark geoRge's shifted-feature detection since both rely entirely on XCMS parameters for feature selection)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000)
```

## Evaluation signals

- The summary CSV exists, is valid CSV, and contains a count row/statistic for both glucose conditions (normal and high glucose) with nonzero counts consistent with detection of isotopically enriched features
- PuInc_seeker returns features meeting fc.threshold=1.5 and p.value.threshold=0.05 with intensity above PuInc.int.lim=4000
- basepeak_finder returns base peaks within ppm.s=6.5 mass tolerance of UL.atomM=12.0 vs L.atomM=13.003355 with Basepeak.minInt=2000
- Shifted features show the expected signature: new and/or higher-intensity m/z peaks in labeled (CELL_Glc13) vs unlabeled (CELL_Glc12) samples
- Cross-check geoRge results against X(13)CMS outcomes, since both rely entirely on XCMS parameters for feature selection

## Limitations

- Results depend entirely on XCMS parameters for feature selection, so peak-picking/alignment settings drive the detected feature set (basis of the X(13)CMS comparison)
- Requires biologically equivalent unlabeled and labeled samples; the ULtag/Ltag class names must match the actual class-name prefix (sep.pos='f')
- geoRge is an untargeted presence/relative-quantification detector; annotation of hits requires a separate database_query with a compound database and adduct set
- geoRge is distributed as an R script installed from GitHub; README advises raising an issue ticket if problems occur, and v1.0 changed/simplified function arguments

## Evidence
