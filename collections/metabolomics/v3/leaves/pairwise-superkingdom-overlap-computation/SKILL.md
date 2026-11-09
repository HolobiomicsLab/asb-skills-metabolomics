---
name: pairwise-superkingdom-overlap-computation
description: Use when a COCONUT snapshot contains organism annotations and the goal
  is to quantify NP production per taxonomic category and the pairwise sharing of
  NPs between superkingdoms.
when_to_use_negative:
- The input dataset lacks organism/provenance annotations (COCONUT's own provenance
  fields such as organism_parts and locations were nullified in pivot tables), so
  no producer taxonomy can be mapped
- The analysis concerns molecular property distributions (descriptors, ClassyFire
  chemical classes) rather than producer-organism taxonomy
- Data comes from a source-specific collection (e.g., SuperNatural II subset) that
  fails COCONUT quality control and has not been unified into the uniqueNaturalProduct
  table
edam_operation: null
tools:
- name: pandas
  role: Loading the COCONUT PostgreSQL snapshot, filtering to NPs with a recorded
    producing organism, mapping organism taxonomy to the five categories, and building
    the per-category counts and pairwise overlap matrix
- name: matplotlib
  role: Rendering the category distribution as grouped bar charts and the superkingdom
    overlap matrix as a heatmap to mirror Fig. 3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:bc959305018f59f77319fc59fd77f61c4a63c6ef62e2db517b9eaeb0044ffea4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matplotlib
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pairwise-superkingdom-overlap-computation

## Summary

Computes the five-way taxonomic provenance breakdown of COCONUT natural products (plants, bacteria, fungi, animals, marine) and a pairwise superkingdom overlap matrix showing which NPs are attributed to more than one producer group, mirroring the COCONUT provenance analysis (Fig. 3).

## When to use

Use when a COCONUT snapshot contains organism annotations and the goal is to quantify NP production per taxonomic category and the pairwise sharing of NPs between superkingdoms.

## When NOT to use

- The input dataset lacks organism/provenance annotations (COCONUT's own provenance fields such as organism_parts and locations were nullified in pivot tables), so no producer taxonomy can be mapped
- The analysis concerns molecular property distributions (descriptors, ClassyFire chemical classes) rather than producer-organism taxonomy
- Data comes from a source-specific collection (e.g., SuperNatural II subset) that fails COCONUT quality control and has not been unified into the uniqueNaturalProduct table

## Inputs

- PostgreSQL snapshot / SQL dump of COCONUT release 2024 (uniqueNaturalProduct records with producing organism, organism rank/iri fields)
- Mapping of organism taxonomy entries to the five provenance categories (plants, bacteria, fungi, animals, marine)

## Outputs

- Percentage share of NPs per taxonomic category (e.g., plant-only ~65%, animal-origin ~0.5%)
- Pairwise superkingdom overlap matrix (counts of molecules attributed to more than one superkingdom)
- Grouped bar chart of per-category NP distribution and heatmap of superkingdom overlaps (Fig. 3 analogue)

## How to apply

Load the PostgreSQL snapshot of the COCONUT release 2024 with pandas and filter to entries with a recorded producing organism. Map each organism entry to the five source categories (plants, bacteria, fungi, animals, marine) using the organism rank/iri fields carried in the data. Count NPs per category and compute percentage shares, checking against the reported baselines: ~65% of NPs with known producers are plant-only and only ~0.5% are of animal origin. Then build a pairwise superkingdom overlap matrix by counting molecules attributed to more than one superkingdom, and render the distribution as grouped bar charts and the overlaps as a matrix heatmap with matplotlib. Flag any recomputed category share that deviates from the reported values beyond rounding tolerance.

## Related tools

- **pandas** (Loading the COCONUT PostgreSQL snapshot, filtering to NPs with a recorded producing organism, mapping organism taxonomy to the five categories, and building the per-category counts and pairwise overlap matrix)
- **matplotlib** (Rendering the category distribution as grouped bar charts and the superkingdom overlap matrix as a heatmap to mirror Fig. 3)

## Examples

```
df = pd.read_sql('SELECT inchikey, organism_rank, organism_iri FROM uniqueNaturalProduct WHERE organism IS NOT NULL', engine); shares = df.groupby('category')['inchikey'].nunique() / df['inchikey'].nunique() * 100
```

## Evaluation signals

- Plant-only fraction recomputes to approximately 65% of NPs with known producer organisms
- Animal-origin fraction recomputes to approximately 0.5%
- Dominant overlaps are plant–marine and plant–fungi; any other pair ranking above these indicates a mapping or filtering error
- Every counted molecule maps to at least one of the five categories; no unmapped organism rank/iri values remain after the mapping step
- Each overlap-matrix off-diagonal cell counts only molecules attributed to both superkingdoms (i.e., molecules are counted per category membership, not double-subtracted from single-category totals)

## Limitations

- Provenance coverage is incomplete: COCONUT's stated final aim of providing organisms that synthesize NPs and their geographic locations is not yet fully provided, so the analysis is restricted to NPs with a recorded producing organism
- The organism_parts and locations pivot-table fields were nullified/unpopulated, limiting deeper provenance breakdowns
- Values are snapshot-dependent (release 2024); recomputed shares should only be compared to the reported 65%/0.5% figures within rounding tolerance

## Evidence

- [other] Taxonomic provenance workflow (task card): [evidence span withheld — anchor status: not_found]
- [methods] Primary data source for the analysis: "The snapshot of the PostgreSQL database form the COCONUT release 2024 was taken as the primary source"
- [discussion] Provenance data limitation: "The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter."
- [discussion] Unpopulated pivot fields: "nullified organism_parts and locations in the pivot tables for now."
