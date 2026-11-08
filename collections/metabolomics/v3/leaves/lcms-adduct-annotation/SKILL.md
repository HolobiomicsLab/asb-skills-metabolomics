---
name: lcms-adduct-annotation
description: Use when geoRge's PuInc_seeker and basepeak_finder have produced labeled
  base-peak m/z values on an XCMS feature table and you need putative metabolite identities
  with adduct assignments in negative-ion LC/MS.
when_to_use_negative:
- Input is raw mzXML or an XCMS xcmsSet that has not been processed by PuInc_seeker
  and basepeak_finder — the skill expects the geoRge labeled base-peak object, not
  raw features.
- Data are acquired in positive-ion mode and no negative-mode adduct list is available
  — the pipeline as documented uses adducts_negative.txt.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: Provides the database_query function used to match labeled base-peak m/z against
    a metabolite database and negative-mode adducts; also supplies PuInc_seeker and
    basepeak_finder to produce its input
  repo: github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment and grouping that produces the XCMSet feature
    table on which geoRge annotation runs
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:0f8672be0f829b19386fb5925aa7e35ef4798011b48e815cb7a627f73b650c07
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MSnbase
  - R
  - X(13)CMS
  - XCMS
  - devtools
  - geoRge
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# lcms-adduct-annotation

## Summary

Annotate isotopically labeled LC/MS features detected by geoRge's basepeak_finder with putative metabolite identities by matching measured m/z values against a metabolite database and negative-mode adduct masses (e.g. -H, +Cl) within a user-defined ppm tolerance.

## When to use

Use when geoRge's PuInc_seeker and basepeak_finder have produced labeled base-peak m/z values on an XCMS feature table and you need putative metabolite identities with adduct assignments in negative-ion LC/MS.

## When NOT to use

- Input is raw mzXML or an XCMS xcmsSet that has not been processed by PuInc_seeker and basepeak_finder — the skill expects the geoRge labeled base-peak object, not raw features.
- Data are acquired in positive-ion mode and no negative-mode adduct list is available — the pipeline as documented uses adducts_negative.txt.

## Inputs

- geoRge basepeak_finder output object (e.g. s2) built on the mtbls213 XCMS feature table
- Negative-mode adduct list (adducts_negative.txt)
- Metabolite database CSV (ExampleDatabase.csv from geoRge extdata)
- ppm.db mass tolerance parameter

## Outputs

- hits: annotated feature-to-metabolite mappings, each hit containing a database metabolite entry and a negative-mode adduct annotation

## How to apply

Run geoRge's upstream steps first: PuInc_seeker on the XCMSet with sample-group tags (e.g. ULtag='CELL_Glc12', Ltag='CELL_Glc13', sep.pos.front=TRUE) and basepeak_finder with the exact isotope mass pair (UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000) to obtain the labeled base-peak object s2. Load the negative-mode adduct list (adducts_negative.txt) and a metabolite mass database (ExampleDatabase.csv from the geoRge extdata directory) into R. Call database_query(geoRgeR = s2, adducts = negative, db = db): the function matches each labeled base-peak m/z against metabolite masses in the database within the ppm.db mass tolerance and reports the matching metabolite plus the negative-mode adduct that explains the measured m/z. Rationale: labeling shifts m/z by isotope multiples, so m/z alone is insufficient; adduct-aware database matching turns base-peak m/z into chemical hypotheses. Validate by checking each annotated hit contains a database metabolite entry and an adduct annotation.

## Related tools

- **geoRge** (Provides the database_query function used to match labeled base-peak m/z against a metabolite database and negative-mode adducts; also supplies PuInc_seeker and basepeak_finder to produce its input) — github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment and grouping that produces the XCMSet feature table on which geoRge annotation runs) — https://bioconductor.org/packages/release/bioc/html/xcms.html

## Examples

```
hits <- database_query(geoRgeR = s2, adducts = negative, db = db)
```

## Evaluation signals

- The script runs end-to-end and writes hits (hits.csv) with every annotated hit containing both a database metabolite entry and an adduct annotation.
- Each reported metabolite mass must fall within ppm.db tolerance of the labeled base-peak m/z after applying the listed negative-mode adduct mass shift.
- Downstream labeling-derived m/z shifts are consistent with the isotope pair used upstream (UL.atomM=12.0 vs L.atomM=13.003355, ppm.s=6.5).
- All labeled base peaks above Basepeak.minInt=2000 in s2 are either annotated or explicitly unannotated — no silent drops.

## Limitations

- Annotations are limited to metabolites present in the supplied database (ExampleDatabase.csv) and to the adducts in the negative-mode list; novel metabolites or unusual adducts will not be annotated.
- The method assumes the user supplies correct isotope masses (12.0 vs 13.003355) and mass tolerances (ppm.s, ppm.db); mis-set tolerances will propagate into wrong annotations.

## Evidence

- [supplementary] geoRge includes a database_query function: [evidence span withheld — anchor status: not_found]
