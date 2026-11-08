---
name: metabolite-database-mass-matching
description: Use when an XCMS/geoRge basepeak_finder result object (s2) of labeled
  base-peak m/z values exists and you need putative metabolite identities plus adduct
  assignments from a metabolite database.
when_to_use_negative:
- When the goal is detecting isotope labeling itself rather than annotating detected
  labeled features — use PuInc_seeker and basepeak_finder instead
- When the input is already an annotated feature/metabolite table and no m/z-to-database
  matching is needed
- When the data do not originate from an XCMS peak-picking/alignment/grouping structure
  of untargeted LC/MS metabolomics data
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: geoRge
  role: Provides the database_query function that performs the m/z-to-database matching
    against the supplied metabolite database and adduct table
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment and grouping whose output structure feeds
    PuInc_seeker and basepeak_finder before database_query
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:84e9b9e3f93c3ceab0df7207a990949139f80c82c6bb2f356fa896bd74ea3326
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-database-mass-matching

## Summary

Annotates labeled LC/MS base peaks detected by geoRge with putative metabolite identities by matching measured m/z values against a metabolite database within a ppm mass tolerance and reporting the negative-mode adduct (e.g. -H, +Cl) that explains each measured m/z.

## When to use

Use when an XCMS/geoRge basepeak_finder result object (s2) of labeled base-peak m/z values exists and you need putative metabolite identities plus adduct assignments from a metabolite database.

## When NOT to use

- When the goal is detecting isotope labeling itself rather than annotating detected labeled features — use PuInc_seeker and basepeak_finder instead
- When the input is already an annotated feature/metabolite table and no m/z-to-database matching is needed
- When the data do not originate from an XCMS peak-picking/alignment/grouping structure of untargeted LC/MS metabolomics data

## Inputs

- geoRge result object s2 from basepeak_finder (labeled base-peak m/z values)
- Metabolite database CSV (ExampleDatabase.csv from geoRge extdata)
- Negative-mode adduct list (adducts_negative.txt)
- ppm.db mass tolerance parameter

## Outputs

- Hits table mapping detected labeled features to putative metabolites with adduct annotations
- hits.csv containing the annotated feature-to-metabolite mappings including adduct information

## How to apply

First run the upstream geoRge pipeline: PuInc_seeker on the XCMS set with ULtag 'CELL_Glc12', Ltag 'CELL_Glc13' and sep.pos 'f', then basepeak_finder with UL.atomM 12.0, L.atomM 13.003355, ppm.s 6.5 and Basepeak.minInt 2000 to obtain s2. Load the negative-mode adduct table (adducts_negative.txt) and the metabolite database (ExampleDatabase.csv shipped in the package extdata directory via system.file). Call database_query(geoRgeR = s2, adducts = negative, db = db): the function matches each labeled base-peak m/z in s2 against metabolite masses in the database within the ppm.db mass tolerance and reports the matching metabolite together with the negative-mode adduct (e.g. -H, +Cl) explaining the measured m/z. The rationale is that isotopically labeled features whose exact m/z is explained by a known metabolite mass plus a plausible adduct gain a putative identity for downstream relative quantification. Finally write the resulting feature-to-metabolite mappings, including adduct information, to hits.csv.

## Related tools

- **geoRge** (Provides the database_query function that performs the m/z-to-database matching against the supplied metabolite database and adduct table) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment and grouping whose output structure feeds PuInc_seeker and basepeak_finder before database_query) — https://bioconductor.org/packages/release/bioc/html/xcms.html

## Examples

```
hits <- database_query(geoRgeR = s2, adducts = negative, db = db)
```

## Evaluation signals

- The R script runs end-to-end and hits.csv is produced
- Each annotated hit in hits.csv contains both a database metabolite entry and an adduct annotation (e.g. -H, +Cl)
- Every reported match is explained within the ppm.db mass tolerance by the metabolite mass plus the assigned adduct
- The hit list is consistent with the labeled base peaks present in s2 from basepeak_finder (run with UL.atomM 12.0, L.atomM 13.003355, ppm.s 6.5, Basepeak.minInt 2000)
- Annotated hits map only to metabolite entries present in the supplied database CSV

## Limitations

- Assignments are limited to metabolites present in the supplied database; true identities outside the database cannot be recovered
- Matches are putative — a mass within ppm.db tolerance plus an adduct does not constitute structural confirmation
- Requires the full upstream geoRge/XCMS pipeline (PuInc_seeker, basepeak_finder) to have been run correctly; the shipped example uses negative-mode adducts (adducts_negative.txt) and the MTBLS213 dataset

## Evidence
