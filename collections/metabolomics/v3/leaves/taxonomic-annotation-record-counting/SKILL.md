---
name: taxonomic-annotation-record-counting
description: Use when you need to quantify what proportion of COCONUT natural products
  carry organism taxa, geographic origin, or literature-reference annotations, e.g.
  to reproduce coverage statistics like 31.5% (taxa), 15.4% (continent) and 16.6%
  (references).
when_to_use_negative:
- You need provenance of molecule parts or fine-grained organism_parts/location fields
  — the article notes these pivot-table fields were nullified and left unpopulated
- Your input is raw structure files (SMILES/SDF) rather than a restored COCONUT database
  — structure QC, unification and annotation computation happen in the COCONUT compiler
  before counting is possible
- You need up-to-date per-molecule annotation star levels rather than aggregate coverage
  fractions of a specific release
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0091
tools:
- name: MongoDB
  role: Store of all original data, unified NPs and derived information; queried via
    pivot tables to count molecules annotated with taxa, locations and references
  repo: github.com/mSorok/NaturalProductsOnline
- name: COCONUT compiler
  role: Java JAR that curates, processes and stores molecules and annotations in the
    Mongo database the counts are run against
  repo: github.com/mSorok/COCONUT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:37e184b1d16c8663fa3fc2929300d0d0a908432cdcc7ab5d833869f694e038b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COCONUT compiler
  - MongoDB
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# taxonomic-annotation-record-counting

## Summary

Computes annotation-coverage fractions for a COCONUT release by counting molecules linked to organism taxa, continent-level geographic locations, and literature references via MongoDB queries over pivot tables derived from the PostgreSQL snapshot.

## When to use

Use when you need to quantify what proportion of COCONUT natural products carry organism taxa, geographic origin, or literature-reference annotations, e.g. to reproduce coverage statistics like 31.5% (taxa), 15.4% (continent) and 16.6% (references).

## When NOT to use

- You need provenance of molecule parts or fine-grained organism_parts/location fields — the article notes these pivot-table fields were nullified and left unpopulated
- Your input is raw structure files (SMILES/SDF) rather than a restored COCONUT database — structure QC, unification and annotation computation happen in the COCONUT compiler before counting is possible
- You need up-to-date per-molecule annotation star levels rather than aggregate coverage fractions of a specific release

## Inputs

- COCONUT release database dump (mongodump directory) restored into MongoDB, e.g. COCONUT2020-07 or a 2024 PostgreSQL snapshot migrated to MongoDB
- Organism taxa, geographic location (continent level), and literature reference pivot tables inside the restored database
- Total molecule count for the release (695,133 molecules in the article's catalog)

## Outputs

- annotation_coverage.csv with three molecule counts (taxa 134,379; continent-level location 66,068; references 70,730) and their fractions of the total (31.5%, 15.4%, 16.6%)

## How to apply

Restore the COCONUT release dump (a PostgreSQL snapshot from the 2024 release has been loaded into MongoDB) with mongorestore using --noIndexRestore for compatibility, then create the documented indexes on sourceNaturalProduct and uniqueNaturalProduct collections. Query the organism annotation pivot tables to count distinct molecules linked to at least one organism taxa, the location pivot tables at continent level to count molecules with a known geographic origin, and the citation/reference tables to count molecules with at least one literature reference. Divide each count by the total molecule count (695,133 in the article's catalog) to obtain coverage fractions. Validate against the reported values of 31.5% (134,379 with taxa), 15.4% (66,068 with continent-level location) and 16.6% (70,730 with references) within rounding tolerance, and write counts plus fractions to annotation_coverage.csv.

## Related tools

- **MongoDB** (Store of all original data, unified NPs and derived information; queried via pivot tables to count molecules annotated with taxa, locations and references) — github.com/mSorok/NaturalProductsOnline
- **COCONUT compiler** (Java JAR that curates, processes and stores molecules and annotations in the Mongo database the counts are run against) — github.com/mSorok/COCONUT

## Examples

```
mongorestore --port 27022 --db=COCONUT_2021_03 --noIndexRestore .
```

## Evaluation signals

- Count of molecules with at least one organism taxa equals 134,379, i.e. 31.5% of 695,133, within rounding tolerance
- Count of molecules with a known continent-level geographic location equals 66,068, i.e. 15.4% of the total
- Count of molecules with at least one literature reference equals 70,730, i.e. 16.6% of the total
- Denominator check: total cataloged molecules in the restored database equals 695,133 (89,593 non-stereo and 555,897 stereo)
- annotation_coverage.csv contains all three count/fraction pairs and fractions sum coherently as independent coverage proportions (not required to sum to 1)

## Limitations

- Provenance coverage is incomplete by design: COCONUT's stated final aim of providing organisms and their geographic location is 'not yet fully provided', and organism_parts/locations pivot-table fields were nullified
- Counts are release-specific — the article's figures refer to a specific catalog (695,133 molecules, 55,252 organisms, 2,653 geographical locations, 24,272 citations) and will differ for other releases
- Geographic location is only counted at continent level, so finer-resolution provenance is not captured

## Evidence

- [intro] All original data, unified NPs and the derived and calculated information are stored in MongoDB.: "All original data, unified NPs and the derived and calculated information are stored in MongoDB."
- [methods] The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules"
- [methods] The snapshot of the PostgreSQL database form the COCONUT release 2024 was taken as the primary source: "The snapshot of the PostgreSQL database form the COCONUT release 2024 was taken as the primary source"
- [discussion] The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter.: "The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter."
- [readme] mongorestore --db=COCONUT --noIndexRestore .: [evidence span withheld — anchor status: not_found]
