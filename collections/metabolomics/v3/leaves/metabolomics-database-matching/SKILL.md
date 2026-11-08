---
name: metabolomics-database-matching
description: Use when geoRge's PuInc_seeker and basepeak_finder have already produced
  labeled-feature base peaks on an XCMS feature table and the goal is to assign putative
  metabolite identities from a database such as ExampleDatabase.csv.
when_to_use_negative:
- Input is raw mzXML/mzML data that has not yet been processed by XCMS peak picking,
  alignment and grouping — database_query operates on the PuIncR/basepeak_finder output,
  not raw files.
- Features have not been through PuInc_seeker and basepeak_finder; querying the full
  unfiltered XCMS feature table will not target isotopically labeled candidates.
- The target analytes ionize only in positive mode and no appropriate adduct list
  is supplied, since the workflow demonstrates annotation with the negative-mode adduct
  list.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: geoRge
  role: Provides database_query (and upstream PuInc_seeker/basepeak_finder) to match
    labeled-feature base peaks against a metabolite database
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Produces the peak-picked, aligned and grouped feature table on which geoRge
    annotation runs
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Language in which geoRge is written and executed, installed via devtools install_github
  repo: https://github.com/jcapelladesto/geoRge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:873e8400789f795dddc71fb29246a0d09729ddf235ba058b18248ff4f17e1cca
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-database-matching

## Summary

Annotate features flagged as putatively isotopically labeled by geoRge by matching their base-peak masses against a metabolite database over a negative-mode adduct list, using the database_query step that follows PuInc_seeker and basepeak_finder.

## When to use

Use when geoRge's PuInc_seeker and basepeak_finder have already produced labeled-feature base peaks on an XCMS feature table and the goal is to assign putative metabolite identities from a database such as ExampleDatabase.csv.

## When NOT to use

- Input is raw mzXML/mzML data that has not yet been processed by XCMS peak picking, alignment and grouping — database_query operates on the PuIncR/basepeak_finder output, not raw files.
- Features have not been through PuInc_seeker and basepeak_finder; querying the full unfiltered XCMS feature table will not target isotopically labeled candidates.
- The target analytes ionize only in positive mode and no appropriate adduct list is supplied, since the workflow demonstrates annotation with the negative-mode adduct list.

## Inputs

- geoRge basepeak_finder output object (PuIncR result, e.g. s2) derived from an XCMS feature table
- Metabolite database CSV (ExampleDatabase.csv shipped in geoRge extdata)
- Adduct list (negative-mode adducts defined in the workflow)
- XCMS feature table from MTBLS213 mzXML files

## Outputs

- Table of database annotation hits for putatively isotopically labeled features (database_query result)
- Feature-level comparison table of geoRge annotations vs X(13)CMS results (CSV)

## How to apply

First run PuInc_seeker on the XCMS object (e.g. XCMSet=mtbls213, ULtag="CELL_Glc12", Ltag="CELL_Glc13", sep.pos.front=TRUE, fc.threshold=1.5, p.value.threshold=0.05, PuInc.int.lim=4000), then run basepeak_finder on its output with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000 to isolate base peaks of labeled features. Load the metabolite database (system.file("extdata/ExampleDatabase.csv", package="geoRge")) and the negative-mode adduct list, then call database_query(geoRgeR=s2, adducts=negative, db=db) to match measured base-peak m/z values against theoretical adduct masses within the supplied ppm tolerance. The result is a table of putative annotations (hits) for the isotopically enriched features, which can then be compared feature-by-feature against X(13)CMS annotations on the same XCMS feature table.

## Related tools

- **geoRge** (Provides database_query (and upstream PuInc_seeker/basepeak_finder) to match labeled-feature base peaks against a metabolite database) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Produces the peak-picked, aligned and grouped feature table on which geoRge annotation runs) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Language in which geoRge is written and executed, installed via devtools install_github) — https://github.com/jcapelladesto/geoRge

## Examples

```
hits <- database_query(geoRgeR = s2, adducts = negative, db = read.csv(system.file("extdata/ExampleDatabase.csv", package="geoRge")))
```

## Evaluation signals

- A hits table is produced from database_query containing annotations for features that passed PuInc_seeker and basepeak_finder filters (PuInc.int.lim=4000, Basepeak.minInt=2000).
- Matched theoretical adduct masses agree with observed base-peak m/z within the ppm.s=6.5 tolerance.
- The database file loaded is the package's extdata/ExampleDatabase.csv and adducts come from the negative-mode list.
- Annotation results can be cross-checked feature-by-feature against X(13)CMS on the same XCMS-selected features, yielding a CSV agreement/discrepancy table.
- Annotated metabolites are chemically consistent with 13C labeling from the ULtag/Ltag pair (CELL_Glc12 vs CELL_Glc13) samples.

## Limitations

- Annotations are putative only; database_query assigns identities from a reference database (ExampleDatabase.csv) and does not confirm them experimentally.
- Matching is limited by the adduct list supplied (the demonstrated workflow uses negative-mode adducts) and by the database's coverage.
- geoRge and X(13)CMS both depend entirely on XCMS parameters (ppm, peakwidth, mzwid, minfrac, bw tuned by trial and error), so annotation and comparison results are sensitive to those feature-selection settings.

## Evidence
