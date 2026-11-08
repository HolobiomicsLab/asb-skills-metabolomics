---
name: monoisotopic-basepeak-assignment
description: Use when an XCMS-derived feature table already contains putatively labeled
  features (PuInc_seeker output) and each labeled feature needs a monoisotopic base
  peak assigned before adduct/database queries.
when_to_use_negative:
- Upstream PuInc_seeker has not been run — basepeak_finder requires the PuIncR object
  with flagged labeled features, not a raw XCMS peak table
- The goal is de novo isotope annotation from theoretical mass differences over all
  MS signals rather than comparing unlabeled vs labeled biologically equivalent samples
  (geoRge's design premise)
- Data were not processed by XCMS, since geoRge 'runs on the output structure of the
  XCMS package'
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: geoRge
  role: Provides the basepeak_finder function that assigns monoisotopic base peaks
    to putatively labeled features given atom masses and ppm tolerance
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Supplies the upstream peak picking, alignment and grouping output (XCMSet)
    on which basepeak_finder operates
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Runtime environment; basepeak_finder is invoked from the R console on XCMS
    objects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:4e3d180712d5048c288988b8c1d9f0013382e9d60b7ba89d0fb709be0f6313af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# monoisotopic-basepeak-assignment

## Summary

geoRge's basepeak_finder step matches each putatively labeled feature (from PuInc_seeker output) to its monoisotopic base peak in the XCMS feature set, using the exact masses of the unlabeled (12.0) and labeled (13.003355) atoms and a ppm tolerance, enabling isotope-envelope annotation in untargeted LC/MS stable-isotope-labeling experiments.

## When to use

Use when an XCMS-derived feature table already contains putatively labeled features (PuInc_seeker output) and each labeled feature needs a monoisotopic base peak assigned before adduct/database queries.

## When NOT to use

- Upstream PuInc_seeker has not been run — basepeak_finder requires the PuIncR object with flagged labeled features, not a raw XCMS peak table
- The goal is de novo isotope annotation from theoretical mass differences over all MS signals rather than comparing unlabeled vs labeled biologically equivalent samples (geoRge's design premise)
- Data were not processed by XCMS, since geoRge 'runs on the output structure of the XCMS package'

## Inputs

- PuInc_seeker result object (s1) flagging putatively labeled feature pairs (e.g. ULtag='CELL_Glc12', Ltag='CELL_Glc13', sep.pos.front=TRUE)
- XCMS feature/peak set from processed LC/MS mzXML data (e.g. data(mtbls213) from MetaboLights MTBLS213)

## Outputs

- geoRge result object (s2) mapping each putatively labeled feature to its monoisotopic base peak m/z, suitable as input to database_query(geoRgeR = s2, ...)
- Feature-to-basepeak assignment table exportable as CSV (e.g. basepeak_assignments.csv)

## How to apply

Run basepeak_finder on the PuInc_seeker result (s1) together with the XCMS feature set (e.g. data(mtbls213)), supplying the atom masses UL.atomM=12.0 and L.atomM=13.003355 for the unlabeled and labeled isotopes. Constrain the matching with a mass tolerance of ppm.s=6.5 and require the base peak candidate to exceed Basepeak.minInt=2000 intensity. The function searches the XCMS peak table for the unlabeled counterpart peak of each putatively labeled feature given the theoretical isotope mass shift, producing a feature-to-basepeak m/z mapping that feeds downstream steps such as database_query. Correct application means every flagged labeled feature carries a monoisotopic base peak assignment consistent with the 6.5 ppm window and the 2000 minimum-intensity threshold used in the methods example.

## Related tools

- **geoRge** (Provides the basepeak_finder function that assigns monoisotopic base peaks to putatively labeled features given atom masses and ppm tolerance) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Supplies the upstream peak picking, alignment and grouping output (XCMSet) on which basepeak_finder operates) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Runtime environment; basepeak_finder is invoked from the R console on XCMS objects)

## Examples

```
s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000)
```

## Evaluation signals

- Output object/table exists and every putatively labeled feature from s1 carries an assigned monoisotopic base peak m/z
- Assignment parameters used match the specification: UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000
- Assigned base peak masses fall within the 6.5 ppm mass tolerance of the expected unlabeled isotope counterpart
- Exported basepeak_assignments.csv is well-formed CSV with labeled features and monoisotopic m/z values
- Downstream database_query accepts the s2 object and returns hits against ExampleDatabase.csv with the chosen adducts (e.g. negative mode)

## Limitations

- Base peak candidates below Basepeak.minInt=2000 intensity will not be assigned, so low-abundance labeled features may lack a base peak
- Matching is bounded by ppm.s=6.5; features with base peaks outside this mass tolerance remain unassigned
- geoRge relies entirely on XCMS parameters for feature selection, so XCMS peak-picking/alignment quality directly constrains basepeak_finder results

## Evidence
