---
name: geographic-origin-coverage-computation
description: Use when auditing annotation coverage of a natural-products database
  that stores molecules, organism taxa, and geographic locations in MongoDB, and you
  need the fraction of NPs with known continent-level provenance.
when_to_use_negative:
- When the goal is structural or chemical annotation coverage (e.g. molecular-property
  descriptors, Murcko frameworks) rather than provenance (organism taxa, geography,
  references).
- When the dataset already provides an explicit per-molecule coverage annotation,
  e.g. COCONUT's own 5-star annotation-level score, which is computed upstream by
  the annotation-level pipeline rather than from pivot tables.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: MongoDB
  role: Stores the restored COCONUT release (molecules, organism-taxa pivot tables,
    and location pivot tables); queries against these collections produce the molecule
    counts used to compute coverage fractions.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:e51a8765a48f194c3b3f20d8bab2e46a2d2d1e1ffd9b68f1b67b744745d654b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MongoDB
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# geographic-origin-coverage-computation

## Summary

Computes the proportion of natural-product entries in a COCONUT-style MongoDB database whose organism's geographic location is annotated, yielding coverage fractions for continent-level origin data against the total number of catalogued molecules.

## When to use

Use when auditing annotation coverage of a natural-products database that stores molecules, organism taxa, and geographic locations in MongoDB, and you need the fraction of NPs with known continent-level provenance.

## When NOT to use

- When the goal is structural or chemical annotation coverage (e.g. molecular-property descriptors, Murcko frameworks) rather than provenance (organism taxa, geography, references).
- When the dataset already provides an explicit per-molecule coverage annotation, e.g. COCONUT's own 5-star annotation-level score, which is computed upstream by the annotation-level pipeline rather than from pivot tables.

## Inputs

- COCONUT PostgreSQL release snapshot (e.g. release 2024, 695,133 molecules) restored into MongoDB
- MongoDB collections storing organism-taxa and geographic-location pivot tables keyed to molecule entries
- Total molecule count for the release used as the coverage denominator

## Outputs

- annotation_coverage.csv containing molecule counts and fractions for organism taxa, continent-level locations, and literature references
- A coverage fraction for geographic-origin annotation (reported as 15.4% / 66,068 molecules in the cited release)

## How to apply

Load a COCONUT PostgreSQL release snapshot (e.g. the 2024 release, 695,133 molecules) into MongoDB, then query the location pivot tables to count molecules linked to at least one known geographic location at the continent level (expected value 66,068 for the 2020 release). Divide that count by the total number of catalogued molecules and express the result as a fraction (reported as 15.4%). The same pipeline is used to compute organism-taxa coverage (134,379 / 695,133 ≈ 31.5%) and literature-reference coverage (70,730 / 695,133 ≈ 16.6%), so all three fractions should be produced together and written to annotation_coverage.csv. Verification relies on the pivot-table join returning a count whose denominator equals the full molecule catalogue — any unannotated or orphaned pivot rows must not be counted.

## Related tools

- **MongoDB** (Stores the restored COCONUT release (molecules, organism-taxa pivot tables, and location pivot tables); queries against these collections produce the molecule counts used to compute coverage fractions.)

## Examples

```
db.uniqueNaturalProduct.find({ 'locations': { $ne: null } }).count() / db.uniqueNaturalProduct.count()
```

## Evaluation signals

- The denominator of the coverage fraction equals the full catalogue size for the release (e.g. 695,133 molecules).
- The continent-level numerator matches the known reference value for the release being audited (66,068 molecules → 15.4%).
- The same query infrastructure reproduces the companion organism-taxa coverage (134,379 → 31.5%) and literature-reference coverage (70,730 → 16.6%).
- Only pivot-table rows whose location is actually annotated (not nullified or empty) are counted toward the numerator.

## Limitations

- Coverage depends on upstream curation quality: the paper notes that pivot-table fields organism_parts and locations were nullified (left unpopulated) in some tables, so a naive count can over- or under-estimate true coverage.
- Coverage is reported at the continent level only; finer-grained geographic resolution may be incomplete and is not captured by this fraction.
- COCONUT's stated aim of complete provenance — organism and geographic location for every NP — is not yet realised; the reported fractions are therefore a partial snapshot, not a completeness certificate.

## Evidence

- [other] Only 31.5% of NPs have known organism taxa and only 15.4% have known continent-level location.: [evidence span withheld — anchor status: not_found]
- [methods] The database catalogs 695,133 chemical molecules: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules"
- [discussion] Pivot table fields organism_parts and locations were nullified: "nullified organism_parts and locations in the pivot tables for now."
- [intro] All original data, unified NPs and the derived and calculated information are stored in MongoDB.: "All original data, unified NPs and the derived and calculated information are stored in MongoDB."
- [discussion] The final aim of COCONUT is to provide NPs structures and their provenance.: "The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter."
