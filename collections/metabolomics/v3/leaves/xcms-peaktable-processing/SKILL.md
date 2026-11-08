---
name: xcms-peaktable-processing
description: Use when an untargeted LC/MS stable isotope-labeling experiment has already
  been preprocessed by XCMS (peak picking, alignment, grouping) into an xcmsSet with
  paired unlabeled/labeled sample tags, and labeled features must be annotated against
  a metabolite database.
when_to_use_negative:
- The peak table was produced by a tool other than XCMS, since geoRge is designed
  to run on the output structure of the XCMS package
- The samples are not paired unlabeled/labeled biologically equivalent samples from
  a stable isotope-labeling experiment, so no isotopic-distribution comparison is
  meaningful
- Only a raw mzXML/mzML dataset without XCMS preprocessing is available; run XCMS
  peak picking, alignment and grouping first
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: geoRge
  role: Provides PuInc_seeker, basepeak_finder and database_query functions that consume
    the XCMS peak table to detect and annotate stable isotope labeling
  repo: github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream preprocessing of raw LC/MS data (peak picking, alignment, grouping)
    whose xcmsSet output is the required input structure
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Implementation and execution environment; geoRge is installed via devtools::install_github
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:127230bbcb154041fb1719910a7badae3ae7c5c83e548484afbc3d952941d98c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# xcms-peaktable-processing

## Summary

Consumes an XCMS-derived peak table (xcmsSet) and processes it through the geoRge functions PuInc_seeker, basepeak_finder and database_query to detect stable-isotope-labeled features and annotate them with putative metabolites and adducts. It exists because geoRge is written in R and operates directly on the XCMS output structure.

## When to use

Use when an untargeted LC/MS stable isotope-labeling experiment has already been preprocessed by XCMS (peak picking, alignment, grouping) into an xcmsSet with paired unlabeled/labeled sample tags, and labeled features must be annotated against a metabolite database.

## When NOT to use

- The peak table was produced by a tool other than XCMS, since geoRge is designed to run on the output structure of the XCMS package
- The samples are not paired unlabeled/labeled biologically equivalent samples from a stable isotope-labeling experiment, so no isotopic-distribution comparison is meaningful
- Only a raw mzXML/mzML dataset without XCMS preprocessing is available; run XCMS peak picking, alignment and grouping first

## Inputs

- XCMS xcmsSet object (output of XCMS peak picking, alignment and grouping) from untargeted LC/MS data
- Sample tag strings identifying unlabeled (ULtag) and labeled (Ltag) biologically equivalent sample groups
- Negative-mode adduct table (adducts_negative.txt from geoRge extdata)
- Metabolite database as CSV (ExampleDatabase.csv shipped in geoRge extdata)

## Outputs

- PuInc_seeker result object (putatively isotopically enriched m/z pairs)
- basepeak_finder result object (s2) of labeled feature base peaks
- hits table from database_query mapping detected labeled features to putative database metabolites with adduct annotations (writable as hits.csv)

## How to apply

Start from an XCMS xcmsSet (e.g. the mtbls213 example dataset from MetaboLights) and run PuInc_seeker with the unlabeled/labeled sample tags (ULtag 'CELL_Glc12', Ltag 'CELL_Glc13', sep.pos 'f') to find putatively isotopically enriched m/z pairs by comparing isotopic distributions between biologically equivalent samples. Pass its result to basepeak_finder with UL.atomM 12.0, L.atomM 13.003355, ppm.s 6.5 and Basepeak.minInt 2000 to identify the monoisotopic base peaks of labeled features. Then feed the geoRge result object (s2) into database_query(geoRgeR, adducts, db, ppm.db): for each labeled base-peak m/z, the function matches metabolite neutral masses in the database (e.g. ExampleDatabase.csv read via system.file('extdata/ExampleDatabase.csv', package='geoRge')) within the ppm.db tolerance, in combination with negative-mode adducts loaded from adducts_negative.txt, reporting the metabolite and adduct (e.g. -H, +Cl) explaining the measured m/z. The rationale is that isotope-enriched compounds shift their isotopic distribution, producing new and more intense m/z peaks in labeled samples, which geoRge detects relative to unlabeled controls rather than iterating over all signals with theoretical mass differences.

## Related tools

- **geoRge** (Provides PuInc_seeker, basepeak_finder and database_query functions that consume the XCMS peak table to detect and annotate stable isotope labeling) — github.com/jcapelladesto/geoRge
- **XCMS** (Upstream preprocessing of raw LC/MS data (peak picking, alignment, grouping) whose xcmsSet output is the required input structure) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Implementation and execution environment; geoRge is installed via devtools::install_github)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,UL.atomM=12.0,L.atomM=13.003355,ppm.s=6.5,Basepeak.minInt=2000); hits <- database_query(geoRgeR=s2, adducts=negative, db=db)
```

## Evaluation signals

- The script runs end-to-end without errors from XCMS xcmsSet through PuInc_seeker, basepeak_finder and database_query
- hits.csv exists and every annotated hit contains both a database metabolite entry and an adduct annotation (e.g. -H, +Cl)
- Putative labeled pairs from PuInc_seeker are restricted to matched ULtag/Ltag sample groups (e.g. 'CELL_Glc12' vs 'CELL_Glc13') and pass fc.threshold 1.5 and p.value.threshold 0.05
- Base peaks reported by basepeak_finder are consistent with a 13C mass shift of 13.003355 - 12.0 Da per labeled atom within the ppm.s 6.5 tolerance and have intensity above Basepeak.minInt 2000
- Annotated measured m/z values are explainable as database metabolite mass plus a listed negative-mode adduct within the ppm.db tolerance

## Limitations

- Adduct matching depends on the supplied adduct table; the documented example uses negative-mode adducts (adducts_negative.txt), so other ionization modes need a different adduct list
- Annotation quality is bounded by the coverage of the metabolite database used (ExampleDatabase.csv in the package) and the ppm.db tolerance
- geoRge relies entirely on XCMS parameters for feature selection, so XCMS preprocessing quality directly limits detected labeled features
- README notes that geoRge 1.0 changed function signatures ('Same functions, less arguments'); older published argument names such as sep.pos='f' map to new arguments like sep.pos.front=TRUE

## Evidence
