---
name: metabolomics-condition-wise-count-aggregation
description: Use when an XCMS peak table from stable-isotope (13C) labeling experiments
  has been processed with PuInc_seeker and basepeak_finder and you need to count how
  many features show isotopic-distribution shifts per condition.
when_to_use_negative:
- When the input is raw LC/MS mzXML data that has not yet been processed by XCMS peak
  picking, alignment and grouping — run the XCMS pipeline first
- When the goal is compound annotation rather than counting label-incorporated features
  — geoRge's database_query with an adduct list is the wrong step for pure counting
- When isotope labels are not stable isotopes detectable via isotopic-distribution
  shifts (e.g. no labeled biologically equivalent sample is available to compare against)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: R package whose PuInc_seeker and basepeak_finder functions flag features with
    isotopic-distribution shifts between unlabeled and labeled samples; its output
    feature lists are what get aggregated into per-condition counts
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: 'Upstream preprocessing: peak picking, alignment and grouping of LC/MS mzXML
    files to produce the XCMSet feature table that PuInc_seeker consumes'
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Language in which geoRge is written and in which the count aggregation and
    CSV export are performed
  repo: https://github.com/jcapelladesto/geoRge
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:6105a8d8c57634f3f81dddb00ec1a70d55c951e986a251eb64626bf98d7af62b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# metabolomics-condition-wise-count-aggregation

## Summary

Aggregate geoRge PuInc_seeker and basepeak_finder results on XCMS feature tables into per-condition counts of features exhibiting isotopic-distribution shifts, producing summary tables that quantify label incorporation under each biological condition (e.g. normal vs high glucose in MTBLS213 ARPE-19 data).

## When to use

Use when an XCMS peak table from stable-isotope (13C) labeling experiments has been processed with PuInc_seeker and basepeak_finder and you need to count how many features show isotopic-distribution shifts per condition.

## When NOT to use

- When the input is raw LC/MS mzXML data that has not yet been processed by XCMS peak picking, alignment and grouping — run the XCMS pipeline first
- When the goal is compound annotation rather than counting label-incorporated features — geoRge's database_query with an adduct list is the wrong step for pure counting
- When isotope labels are not stable isotopes detectable via isotopic-distribution shifts (e.g. no labeled biologically equivalent sample is available to compare against)

## Inputs

- XCMS XCMSet object or peak table derived from mzXML files (e.g. MTBLS213 LC/MS data)
- Class/tag assignments for unlabeled and labeled sample groups (e.g. CELL_Glc12 vs CELL_Glc13)
- PuInc_seeker output object (PuIncR) containing putative isotopically labelled features

## Outputs

- Per-condition counts of features with isotopic-distribution shifts (normal vs high glucose)
- List of shifted-feature identifiers (m/z, retention time) per condition
- Summary CSV table (e.g. isotope_shift_features_summary.csv)

## How to apply

First process the mzXML files (e.g. MTBLS213) with XCMS peak picking, alignment and grouping to obtain an XCMSet feature table with class tags such as CELL_Glc12 (unlabeled) and CELL_Glc13 (13C-labeled glucose). Run geoRge's PuInc_seeker with ULtag='CELL_Glc12', Ltag='CELL_Glc13', sep.pos.front=TRUE, fc.threshold=1.5, p.value.threshold=.05 and PuInc.int.lim=4000 to flag features whose m/z peaks are new or significantly more intense in labeled versus unlabeled biologically equivalent samples. Then run basepeak_finder on the PuIncR output with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to refine the shifted-feature list via their base peaks. Finally, aggregate the putative-incorporation and basepeak features to count how many exhibit the isotopic-distribution shift per condition (normal vs high glucose) and write the counts and feature identifiers to a summary CSV (isotope_shift_features_summary.csv). Validate that the table is valid CSV, contains a count row for both glucose conditions, and that counts are nonzero and consistent with geoRge's detection of isotopically enriched features.

## Related tools

- **geoRge** (R package whose PuInc_seeker and basepeak_finder functions flag features with isotopic-distribution shifts between unlabeled and labeled samples; its output feature lists are what get aggregated into per-condition counts) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream preprocessing: peak picking, alignment and grouping of LC/MS mzXML files to produce the XCMSet feature table that PuInc_seeker consumes) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Language in which geoRge is written and in which the count aggregation and CSV export are performed) — https://github.com/jcapelladesto/geoRge

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,UL.atomM=12.0,L.atomM=13.003355,ppm.s=6.5,Basepeak.minInt=2000)
```

## Evaluation signals

- Output CSV parses as valid CSV and contains a count row/statistic for both glucose conditions (e.g. CELL_Glc12/normal and CELL_Glc13/high glucose)
- Counts are nonzero and consistent with geoRge's reported detection of isotopically enriched features
- Aggregate feature counts match the cardinality of the PuInc_seeker output (PuIncR) filtered by the basepeak_finder refinement (ppm.s=6.5, Basepeak.minInt=2000)
- Per-condition counts differ between conditions, reflecting the different isotopic-distribution shifts under normal vs high glucose mimicking diabetic retinopathy in vitro
- Feature identifiers in the summary table can be traced back to rows in the XCMSet feature table

## Limitations

- Counts depend entirely on XCMS parameters for feature selection — geoRge (and thus the counts) relies on XCMS's peak picking/alignment/grouping settings
- Detection thresholds (fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000, ppm.s=6.5, Basepeak.minInt=2000) directly bound which features are counted; changing them changes the counts
- Requires biologically equivalent unlabeled and labeled samples (ULtag/Ltag) to compare isotopic distributions; cannot enumerate label incorporation without such paired conditions

## Evidence
