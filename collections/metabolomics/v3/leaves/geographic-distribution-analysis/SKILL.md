---
name: geographic-distribution-analysis
description: Use when a natural product dataset carries organism geographic-location
  provenance and the goal is to identify which continents dominate (or are absent/sparse
  among) provenance records and how NPs are shared across continents.
when_to_use_negative:
- The task is chemical-structure QC or curation (e.g. 5–210 heavy atom size checks,
  polyfluorinated compound removal, duplicate merging) — use the COCONUT curation/filter
  pipeline instead.
- The input records lack organism geographic-location annotations entirely (COCONUT
  itself notes locations in pivot tables were 'nullified ... for now', so provenance
  analysis is only meaningful for the subset with known locations).
- The goal is molecular descriptor, fingerprint, or substructure/similarity search
  rather than provenance aggregation.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: PostgreSQL
  role: Source of the COCONUT release 2024 snapshot taken as the primary data source
    for provenance records
- name: MongoDB
  role: Stores the COCONUT data dump (sourceNaturalProduct / uniqueNaturalProduct
    collections) that can be restored locally with mongorestore for provenance querying
- name: pandas
  role: Aggregates natural product counts per continent of provenance from the provenance
    records
- name: matplotlib
  role: Renders the UpSet-style continent-overlap visualization highlighting Asia's
    dominance and the absence of Australian data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:d12071ce2858e2b6466ce4227e751770e2e5c0d8e1ecfac955d27b1938974f31
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MongoDB
  - PostgreSQL
  - matplotlib
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# geographic-distribution-analysis

## Summary

Aggregates natural product provenance records (molecule-organism-location) by continent of organism isolation and computes continent-level overlap of shared NPs, reproducing the Fig. 4 analysis of COCONUT that reveals Asia's dominance and Africa's strong representation among geographically annotated NPs.

## When to use

Use when a natural product dataset carries organism geographic-location provenance and the goal is to identify which continents dominate (or are absent/sparse among) provenance records and how NPs are shared across continents.

## When NOT to use

- The task is chemical-structure QC or curation (e.g. 5–210 heavy atom size checks, polyfluorinated compound removal, duplicate merging) — use the COCONUT curation/filter pipeline instead.
- The input records lack organism geographic-location annotations entirely (COCONUT itself notes locations in pivot tables were 'nullified ... for now', so provenance analysis is only meaningful for the subset with known locations).
- The goal is molecular descriptor, fingerprint, or substructure/similarity search rather than provenance aggregation.

## Inputs

- COCONUT release 2024 PostgreSQL snapshot with molecule-organism-location provenance records
- COCONUT MongoDB dump (e.g. COCONUT_2021_03) containing sourceNaturalProduct and uniqueNaturalProduct collections
- Subset of NPs with known organism geographic location (COCONUT catalogs 55,252 organisms and 2,653 geographical locations)

## Outputs

- Continent-level provenance counts (natural product counts per continent of organism isolation)
- Continent-level provenance overlap statistics (sets of NPs shared across continents)
- UpSet-style overlap figure saved as fig4_provenance_continents.png

## How to apply

Take a snapshot of the PostgreSQL database from the COCONUT release 2024 as the primary source (or restore the MongoDB dump via mongorestore without indexes, then re-create indexes such as simpleInchiKey and source). Load the molecule-organism-location provenance records, subset to NPs whose organism geographic location is known, and use pandas to aggregate counts per continent of provenance. Then compute continent-level provenance-overlap statistics — the sets of NPs shared across continents — following the Fig. 4 methodology, and render an UpSet-style overlap visualization with matplotlib. The expected structure of the result is Asia dominating (attributed to intensive study of traditional Chinese and Indian medicines), Africa strongly represented, Australia absent, and Europe sparsely represented.

## Related tools

- **PostgreSQL** (Source of the COCONUT release 2024 snapshot taken as the primary data source for provenance records)
- **MongoDB** (Stores the COCONUT data dump (sourceNaturalProduct / uniqueNaturalProduct collections) that can be restored locally with mongorestore for provenance querying)
- **pandas** (Aggregates natural product counts per continent of provenance from the provenance records)
- **matplotlib** (Renders the UpSet-style continent-overlap visualization highlighting Asia's dominance and the absence of Australian data)

## Examples

```
mongorestore --port 27022 --db=COCONUT_2021_03 --noIndexRestore .
```

## Evaluation signals

- The output figure fig4_provenance_continents.png exists and is a valid PNG.
- Continent ordering matches the reported Fig. 4 trend: Asia dominant, Africa strong, Australia absent, Europe sparse.
- Counts are restricted to the subset of NPs with known organism geographic location, not the full 695,133-molecule catalog.
- Overlap statistics partition NPs by shared-continent membership so absence/sparse patterns (Australia absent, Europe sparse) are visible in the UpSet-style layout.
- Expert review confirms the reproduction matches the reported Fig. 4 presence/absence patterns.

## Limitations

- Provenance data is COCONUT's 'final aim' not yet fully provided; organism_parts and locations in the pivot tables were 'nullified ... for now', so geographic annotation coverage is incomplete.
- The Asia dominance reflects a literature/collection bias (intensive study of traditional Chinese and Indian medicines), not a true global biological distribution, so counts should not be read as biodiversity estimates.
- Only the subset of NPs with known geographic location can be analyzed; NPs from the 63 sources lacking location annotation are excluded.

## Evidence

- [methods] Primary source is COCONUT release 2024 PostgreSQL snapshot: "The snapshot of the PostgreSQL database form the COCONUT release 2024 was taken as the primary source"
- [discussion] COCONUT aims to provide provenance including geographic location: "The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter."
- [discussion] Locations in pivot tables left unpopulated: "nullified organism_parts and locations in the pivot tables for now."
- [readme] Data restored to MongoDB without indexes: [evidence span withheld — anchor status: external_doc]
- [methods] Database catalogs organisms and geographical locations: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules"
