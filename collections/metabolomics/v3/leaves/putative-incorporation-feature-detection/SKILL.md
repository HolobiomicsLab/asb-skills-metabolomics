---
name: putative-incorporation-feature-detection
description: Use when an XCMS feature table has biologically equivalent unlabeled
  vs stable-isotope-labeled LC/MS sample classes (e.g., 12C vs 13C glucose) and you
  must flag features that are new or higher-intensity in labeled samples.
when_to_use_negative:
- Input is raw mzXML/NetCDF not yet processed by XCMS — geoRge runs on the XCMS output
  structure, so peak picking, alignment and grouping must happen first.
- There are no biologically equivalent unlabeled/labeled sample pairs to contrast;
  geoRge compares paired isotopic distributions rather than iterating over all MS
  signals using theoretical mass differences.
- Expected labeled features fall below the intensity gates (PuInc.int.lim=4000, Basepeak.minInt=2000)
  or the basepeak match exceeds the 6.5 ppm tolerance, so the incorporation signal
  cannot be detected.
edam_topics:
- http://edamontology.org/topic_3344
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: 'Core implementation: PuInc_seeker flags features whose isotopic distribution
    shifts between ULtag/Ltag samples; basepeak_finder then identifies the labeled
    base peaks.'
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: 'Upstream engine: performs peak picking, alignment and grouping on mzXML files
    to produce the XCMSet feature table that geoRge consumes.'
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: 'Runtime environment: geoRge is written in the open language R and installed
    from GitHub with devtools::install_github(''jcapelladesto/geoRge'').'
- name: X(13)CMS
  role: 'Comparator: geoRge''s detected labeling features were cross-checked against
    X(13)CMS outcomes since both rely entirely on XCMS parameters for feature selection.'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:74834277449faf778dd38671e44d06dd32d1b4424570e588277d3fb2795b08d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - X(13)CMS
  - XCMS
  - geoRge
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3344
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# putative-incorporation-feature-detection

## Summary

geoRge-based detection of putative stable-isotope (13C) incorporation in untargeted LC/MS metabolomics: PuInc_seeker and basepeak_finder compare isotopic distributions between biologically equivalent unlabeled and labeled samples on an XCMS feature table to flag features with new or higher-intensity m/z peaks in the labeled condition. On MTBLS213 (ARPE-19 cells grown on normal vs high glucose), this enumerates the features whose isotopic distribution shifts per glucose condition.

## When to use

Use when an XCMS feature table has biologically equivalent unlabeled vs stable-isotope-labeled LC/MS sample classes (e.g., 12C vs 13C glucose) and you must flag features that are new or higher-intensity in labeled samples.

## When NOT to use

- Input is raw mzXML/NetCDF not yet processed by XCMS — geoRge runs on the XCMS output structure, so peak picking, alignment and grouping must happen first.
- There are no biologically equivalent unlabeled/labeled sample pairs to contrast; geoRge compares paired isotopic distributions rather than iterating over all MS signals using theoretical mass differences.
- Expected labeled features fall below the intensity gates (PuInc.int.lim=4000, Basepeak.minInt=2000) or the basepeak match exceeds the 6.5 ppm tolerance, so the incorporation signal cannot be detected.

## Inputs

- XCMS XCMSet feature table produced by peak picking, alignment and grouping of LC/MS mzXML files (e.g., MetaboLights MTBLS213)
- Class-name tags identifying biologically equivalent unlabeled and labeled sample groups (ULtag='CELL_Glc12', Ltag='CELL_Glc13')
- PuInc_seeker output object (PuIncR), consumed by basepeak_finder
- Optional metabolite database CSV (geoRge extdata/ExampleDatabase.csv) with adduct definitions for database_query

## Outputs

- PuIncR object (s1): features flagged as putatively isotopically labeled, with fold-change and p-value statistics
- Basepeak results (s2): base peaks of labeled features matched by the 12.0 to 13.003355 Da mass difference within 6.5 ppm
- Per-condition summary table (e.g., isotope_shift_features_summary.csv) with counts and identifiers of isotopic-shift features for each glucose condition

## How to apply

Build an XCMSet by running XCMS peak picking, alignment and grouping on the LC/MS mzXML files (e.g., MTBLS213), keeping class names that encode label status behind a common prefix split by the position argument (sep.pos='f' in geoRge 1.0; sep.pos.front=TRUE in earlier releases) with ULtag='CELL_Glc12' and Ltag='CELL_Glc13'. Run PuInc_seeker on this XCMSet with fc.threshold=1.5, p.value.threshold=0.05 and PuInc.int.lim=4000: it compares isotopic distributions between biologically equivalent unlabeled and labeled samples and flags features whose m/z peaks are new or significantly more intense in labeled samples, i.e., putative 13C incorporation. Refine the list with basepeak_finder (PuIncR=s1, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000) to identify the base peaks of the putatively labeled features. Optionally annotate surviving hits with database_query against a metabolite database such as the package's extdata/ExampleDatabase.csv using negative-mode adducts. Finally aggregate the flagged features per condition (normal vs high glucose) into a summary table of counts and feature identifiers; the rationale is that isotopically enriched compounds change their isotopic distribution, directly creating new and higher-intensity m/z peaks relative to unlabeled controls.

## Related tools

- **geoRge** (Core implementation: PuInc_seeker flags features whose isotopic distribution shifts between ULtag/Ltag samples; basepeak_finder then identifies the labeled base peaks.) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream engine: performs peak picking, alignment and grouping on mzXML files to produce the XCMSet feature table that geoRge consumes.) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Runtime environment: geoRge is written in the open language R and installed from GitHub with devtools::install_github('jcapelladesto/geoRge').)
- **X(13)CMS** (Comparator: geoRge's detected labeling features were cross-checked against X(13)CMS outcomes since both rely entirely on XCMS parameters for feature selection.)

## Examples

```
library(geoRge); s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000)
```

## Evaluation signals

- PuInc_seeker returns a non-empty PuIncR set and per-condition counts of shifted features are nonzero for both glucose conditions (CELL_Glc12 vs CELL_Glc13).
- Flagged features satisfy the configured gates: fold change >= 1.5 (fc.threshold), p <= 0.05 (p.value.threshold), and intensity >= 4000 (PuInc.int.lim).
- basepeak_finder matches labeled base peaks to the 13.003355 - 12.0 = 1.003355 Da mass difference within ppm.s = 6.5 at intensity >= Basepeak.minInt = 2000.
- The summary output (isotope_shift_features_summary.csv) exists, parses as valid CSV, and contains a count row/statistic for each glucose condition.
- Detected features are broadly consistent with an X(13)CMS run on the same XCMSet, since both approaches rely entirely on XCMS parameters for feature selection.

## Limitations

- Requires biologically equivalent unlabeled and labeled samples; it cannot be applied when only one condition exists.
- Results rely entirely on XCMS parameters for feature selection, so peak picking/alignment/grouping quality bounds the labeling call set.
- Sensitivity is bounded by the configured gates (fc.threshold=1.5, p.value.threshold=0.05, PuInc.int.lim=4000, Basepeak.minInt=2000, ppm.s=6.5); low-intensity or weakly enriched features may be missed.
- Class-name parsing is error-prone: the README uses sep.pos.front=TRUE while geoRge 1.0 / the article methods use sep.pos='f' — misconfiguration misassigns unlabeled/labeled samples.
- The geoRge 1.0 changelog notes that label_compare() was patched, indicating a prior bug in that function; problems should be raised as GitHub issue tickets.

## Evidence

- [readme] s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",
sep.pos.front=TRUE ,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim = 4000): "s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",
sep.pos.front=TRUE ,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim = 4000)"
- [readme] s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0,L.atomM=13.003355,
	ppm.s=6.5,Basepeak.minInt=2000): "s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0,L.atomM=13.003355,
	ppm.s=6.5,Basepeak.minInt=2000)"
