---
name: metabolomics-data-table-export
description: Use when PuInc_seeker (ULtag='CELL_Glc12', Ltag='CELL_Glc13') and basepeak_finder
  (UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000) results exist
  and a flat CSV of labeled features is the deliverable.
when_to_use_negative:
- Input is still raw mzXML files that have not been processed by XCMS peak picking,
  alignment and grouping — no annotation table exists yet to export
- PuInc_seeker or basepeak_finder has not been run (there is no s1/s2 result object);
  exporting directly from the XCMS grouped feature table would miss the isotope-pair
  annotations
- The analysis goal is isotope-annotation itself rather than tabulating existing results
  (use geoRge's PuInc_seeker/basepeak_finder/database_query steps instead)
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: geoRge
  role: Produces the s1 (PuInc_seeker) and s2 (basepeak_finder) result tables that
    are exported to CSV
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Supplies the upstream peak-picked, aligned and grouped feature table (mtbls213
    XCMSet) whose columns populate the exported table
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Language in which geoRge runs and in which write.csv exports the final isotope
    table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:c4907cb91c59029c5e47005ce1277ea99a8a2fa332143b386bb112503f318c35
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolomics-data-table-export

## Summary

Export the geoRge stable-isotope-annotation results (PuInc_seeker + basepeak_finder output tables) as a machine-readable CSV, e.g. george_mtbls213_isotope_table.csv, so that putative isotopically enriched features with base-peak isotope assignments can be reused downstream.

## When to use

Use when PuInc_seeker (ULtag='CELL_Glc12', Ltag='CELL_Glc13') and basepeak_finder (UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000) results exist and a flat CSV of labeled features is the deliverable.

## When NOT to use

- Input is still raw mzXML files that have not been processed by XCMS peak picking, alignment and grouping — no annotation table exists yet to export
- PuInc_seeker or basepeak_finder has not been run (there is no s1/s2 result object); exporting directly from the XCMS grouped feature table would miss the isotope-pair annotations
- The analysis goal is isotope-annotation itself rather than tabulating existing results (use geoRge's PuInc_seeker/basepeak_finder/database_query steps instead)

## Inputs

- s2 data frame from geoRge basepeak_finder (run on s1 from PuInc_seeker with XCMSet=mtbls213)
- optional hits data frame from geoRge database_query (adducts=negative, db=ExampleDatabase.csv)
- XCMS feature table from the MTBLS213 ARPE-19 LC/MS dataset (pre-built mtbls213 object or mzXML-derived)

## Outputs

- george_mtbls213_isotope_table.csv containing putative isotopically enriched features with base-peak isotope assignments and labeled-vs-unlabeled intensity ratios

## How to apply

First ensure the XCMS feature table (peak picking, alignment and grouping on the MTBLS213 mzXML files, or the pre-built mtbls213 object shipped with geoRge) has been run through PuInc_seeker and then basepeak_finder. Take the resulting data frame (s2 from basepeak_finder, optionally extended with database_query hits using the negative-mode adducts and ExampleDatabase.csv) and write it to CSV, preserving the per-feature labeled-vs-unlabeled intensity ratios and base-peak assignments. In R use write.csv on the basepeak_finder result object to produce george_mtbls213_isotope_table.csv. Verify the export by re-reading the CSV and checking that the row count matches nrow(s2) and that enrichment ratios reflect the fc.threshold=1.5 filter used in PuInc_seeker (p.value.threshold=0.05, PuInc.int.lim=4000).

## Related tools

- **geoRge** (Produces the s1 (PuInc_seeker) and s2 (basepeak_finder) result tables that are exported to CSV) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Supplies the upstream peak-picked, aligned and grouped feature table (mtbls213 XCMSet) whose columns populate the exported table) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Language in which geoRge runs and in which write.csv exports the final isotope table)

## Examples

```
s2 <- basepeak_finder(PuIncR=s1, XCMSet=mtbls213, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000); write.csv(s2, file="george_mtbls213_isotope_table.csv", row.names=FALSE)
```

## Evaluation signals

- The CSV file parses cleanly (e.g. read.csv in R or pandas.read_csv) and its row count equals nrow(s2) from basepeak_finder
- Columns include the PuInc_seeker/basepeak_finder outputs: labeled-vs-unlabeled intensity ratio and base-peak isotope assignments computed with UL.atomM=12.0 vs L.atomM=13.003355 at ppm.s=6.5
- Every retained feature passed the PuInc_seeker filters (fc.threshold=1.5, p.value.threshold=0.05, PuInc.int.lim=4000), so exported ratios should be >=1.5
- Base-peak intensities in the table respect Basepeak.minInt=2000
- Re-deriving s2 from mtbls213 with the README's example calls reproduces the same rows in the exported CSV (stable diff)

## Limitations

- geoRge runs on the output structure of XCMS, so the export is only as good as the upstream XCMS peak picking, alignment and grouping parameters
- Results are putative isotopically enriched features, not confirmed tracer-derived metabolites; the table should not be treated as definitive isotope assignments
- Older geoRge versions use different argument names (e.g. sep.pos vs sep.pos.front), so exports may vary across package versions

## Evidence
