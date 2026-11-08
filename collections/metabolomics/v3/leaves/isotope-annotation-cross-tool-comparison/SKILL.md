---
name: isotope-annotation-cross-tool-comparison
description: Use when two isotope-labeling detection tools (e.g. geoRge and X(13)CMS)
  have been run on an identical XCMS feature table and you need a per-feature overlap/discrepancy
  comparison of their labeled-feature calls.
when_to_use_negative:
- The two tools were run on different XCMS parameter sets or different feature tables
  — the comparison is only justified when both rely on identical XCMS m/z and retention-time
  feature-selection parameters
- The goal is de novo isotope annotation rather than benchmarking/cross-checking labeling-detection
  tools against each other
- Input data is not paired unlabeled/labeled samples (e.g. no ULtag/Ltag sample groups
  such as CELL_Glc12 vs CELL_Glc13), so PuInc_seeker cannot define putative labeled
  pairs
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: geoRge
  role: Detects putative isotopically-labeled pairs (PuInc_seeker), base peaks (basepeak_finder)
    and annotations (database_query) on the XCMS feature table; one side of the comparison
  repo: https://github.com/jcapelladesto/geoRge
- name: X(13)CMS
  role: Alternative isotope-labeling detection tool run on the same XCMS feature table;
    provides the reference outcome to diff against geoRge
- name: XCMS
  role: Performs peak picking, alignment and grouping to produce the shared feature
    table that guarantees a fair feature-level comparison
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: devtools
  role: Installs geoRge from GitHub into R (install_github('jcapelladesto/geoRge'))
  repo: https://github.com/jcapelladesto/geoRge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:8883ef65554c12b371198818ef468d353b421421ffbaccd4ab727864de9d658e
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# isotope-annotation-cross-tool-comparison

## Summary

Compares stable-isotope-labeled features detected by geoRge against X(13)CMS on the same XCMS feature table (MTBLS213), producing a feature-level agreement/discrepancy table justified because both tools rely entirely on XCMS parameters for feature selection.

## When to use

Use when two isotope-labeling detection tools (e.g. geoRge and X(13)CMS) have been run on an identical XCMS feature table and you need a per-feature overlap/discrepancy comparison of their labeled-feature calls.

## When NOT to use

- The two tools were run on different XCMS parameter sets or different feature tables — the comparison is only justified when both rely on identical XCMS m/z and retention-time feature-selection parameters
- The goal is de novo isotope annotation rather than benchmarking/cross-checking labeling-detection tools against each other
- Input data is not paired unlabeled/labeled samples (e.g. no ULtag/Ltag sample groups such as CELL_Glc12 vs CELL_Glc13), so PuInc_seeker cannot define putative labeled pairs

## Inputs

- XCMS feature table (xcmSet object) built from MTBLS213 mzXML files with identical m/z and retention-time parameters for both tools
- geoRge PuInc_seeker / basepeak_finder / database_query outputs (PuIncR, base-peak hits) with parameters UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000
- X(13)CMS output on the same XCMS feature table
- ExampleDatabase.csv and negative-mode adduct list from the geoRge package extdata

## Outputs

- CSV agreement table containing per-feature overlap and discrepancy entries between geoRge and X(13)CMS annotations for MTBLS213

## How to apply

Build a single XCMS peak table (peak picking, alignment, grouping with tuned ppm, peakwidth, mzwid, minfrac, bw) from the MTBLS213 mzXML files so both tools consume identical m/z and retention-time features. Run geoRge's PuInc_seeker (ULtag='CELL_Glc12', Ltag='CELL_Glc13', sep.pos='f'), then basepeak_finder (UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000) and database_query against the negative-mode adduct list and ExampleDatabase.csv to obtain geoRge annotations. Run X(13)CMS on the same XCMSet object and diff its labeled-feature calls against geoRge's at feature level, recording shared hits and discrepancies. The comparison is valid precisely because both approaches rely entirely on XCMS parameters for feature selection, so differences are attributable to the labeling-detection algorithm rather than upstream feature definition.

## Related tools

- **geoRge** (Detects putative isotopically-labeled pairs (PuInc_seeker), base peaks (basepeak_finder) and annotations (database_query) on the XCMS feature table; one side of the comparison) — https://github.com/jcapelladesto/geoRge
- **X(13)CMS** (Alternative isotope-labeling detection tool run on the same XCMS feature table; provides the reference outcome to diff against geoRge)
- **XCMS** (Performs peak picking, alignment and grouping to produce the shared feature table that guarantees a fair feature-level comparison) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **devtools** (Installs geoRge from GitHub into R (install_github('jcapelladesto/geoRge'))) — https://github.com/jcapelladesto/geoRge

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000)
```

## Evaluation signals

- An agreement table exists, is in CSV format, and contains per-feature overlap and discrepancy entries between geoRge and X(13)CMS annotations for MTBLS213
- Both tools were demonstrably run on the same XCMS feature table (identical ppm, peakwidth, mzwid, minfrac, bw parameters) before comparison
- geoRge pipeline parameters match the README example: fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000, ppm.s=6.5, Basepeak.minInt=2000
- Shared hits and discrepancies are recorded at feature level (m/z / retention-time feature identity), not just as aggregate counts
- Database annotations come from database_query against the negative-mode adduct list and ExampleDatabase.csv shipped with geoRge

## Limitations

- Both tools depend entirely on XCMS parameters for feature selection; the paper notes XCMS parameters (ppm, peakwidth, mzwid, minfrac, bw) were tuned by trial and error, so comparisons are conditional on those choices
- Comparison is feature-level on MTBLS213 LC/MS data; conclusions may not transfer to other datasets, labeling elements, or adduct modes
- geoRge label_compare() had a bug patched in version 1.0 per the README; ensure the updated package version is used

## Evidence

- [abstract] geoRge results were compared with X(13)CMS outcomes since both rely entirely on XCMS parameters for feature selection: "we compared the results of geoRge with the outcome of X(13)CMS, since both approaches rely entirely on XCMS parameters for feature selection"
- [methods] XCMS provides the shared feature table: "Use XCMS package (https://bioconductor.org/packages/release/bioc/html/xcms.html) for peak picking, alignment and grouping"
- [methods] geoRge pipeline calls with concrete parameters: "s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13","
- [readme] README example invocation with thresholds: "s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0,L.atomM=13.003355,
	ppm.s=6.5,Basepeak.minInt=2000)"
- [readme] label_compare patched in geoRge 1.0: [evidence span withheld — anchor status: external_doc]
