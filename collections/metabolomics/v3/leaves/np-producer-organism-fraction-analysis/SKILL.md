---
name: np-producer-organism-fraction-analysis
description: Use when analyzing a natural products dataset with recorded producing
  organisms and you need per-category producer fractions (e.g., plants ~65%, animals
  ~0.5%) plus cross-category production overlap counts.
when_to_use_negative:
- The dataset entries lack recorded producing organism fields, so no provenance-based
  category assignment is possible
- The goal is chemical structure analysis (descriptors, Murcko frameworks, fingerprints)
  rather than producer-organism taxonomy distribution
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3174
tools:
- name: pandas
  role: Loads the COCONUT 2024 PostgreSQL snapshot, filters to entries with a recorded
    producing organism, maps organism ranks/IRIs to the five source categories, and
    computes counts and percentage shares
- name: matplotlib
  role: Renders the per-category NP distribution as grouped bar charts and the superkingdom
    overlap matrix as a heatmap to mirror Fig. 3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:d91e74ce4dc13274881322d887296e115d6c36f180c6049642865a3e4530f8c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# np-producer-organism-fraction-analysis

## Summary

Quantifies how natural products with a known producing organism are distributed across the five taxonomic source categories (plants, bacteria, fungi, animals, marine) in the COCONUT database, and computes pairwise superkingdom overlap of NP production.

## When to use

Use when analyzing a natural products dataset with recorded producing organisms and you need per-category producer fractions (e.g., plants ~65%, animals ~0.5%) plus cross-category production overlap counts.

## When NOT to use

- The dataset entries lack recorded producing organism fields, so no provenance-based category assignment is possible
- The goal is chemical structure analysis (descriptors, Murcko frameworks, fingerprints) rather than producer-organism taxonomy distribution

## Inputs

- COCONUT PostgreSQL database snapshot (release 2024 export) loaded as a pandas DataFrame
- Organism taxonomy annotations with organism rank/iri fields for entries having a recorded producing organism

## Outputs

- Percentage share of NPs per taxonomic source category (plants, bacteria, fungi, animals, marine)
- Pairwise superkingdom overlap matrix counting shared NP production
- Grouped bar chart of category distribution and matrix heatmap of superkingdom overlaps (mirroring Fig. 3)

## How to apply

Load the COCONUT PostgreSQL release 2024 snapshot with pandas and retain only entries having a recorded producing organism, using the organism rank/iri fields to map each producer to the five source categories (plants, bacteria, fungi, animals, marine). Count NPs per category, compute percentage shares, and compare the recomputed fractions against reported anchors (plants ~65%, animal origin ~0.5%) within rounding tolerance. Build a pairwise superkingdom overlap matrix that counts molecules attributed to more than one superkingdom, expecting plant-marine and plant-fungi as the dominant overlaps. Render the distribution as grouped bar charts and the overlaps as a matrix heatmap with matplotlib to mirror the paper's Fig. 3; flag any category whose share deviates beyond rounding tolerance.

## Related tools

- **pandas** (Loads the COCONUT 2024 PostgreSQL snapshot, filters to entries with a recorded producing organism, maps organism ranks/IRIs to the five source categories, and computes counts and percentage shares)
- **matplotlib** (Renders the per-category NP distribution as grouped bar charts and the superkingdom overlap matrix as a heatmap to mirror Fig. 3)

## Examples

```
import pandas as pd; df = pd.read_sql("SELECT * FROM uniqueNaturalProduct WHERE organism IS NOT NULL", con); df['category'] = df['organism_iri'].map(map_to_five_categories); shares = df['category'].value_counts(normalize=True) * 100
```

## Evaluation signals

- Plant-only share recomputes to approximately 65% of NPs with known producer organisms
- Animal-origin share recomputes to approximately 0.5%
- Plant-marine and plant-fungi emerge as the dominant pairwise superkingdom overlaps
- Any category whose recomputed share deviates beyond rounding tolerance from the reported value is flagged
- Overlap matrix is symmetric with zero diagonal counts of molecules attributed to only one superkingdom counted as overlaps

## Limitations

- Provenance data (organisms synthesizing NPs and their geographic locations) is a stated final aim of COCONUT not yet fully provided; organism_parts and locations pivot table fields were nullified
- Only NPs with a known provenance organism can be classified, so NPs lacking producer annotations are excluded from the fractions
- Results depend on the COCONUT release version; the analysis is anchored to the PostgreSQL release 2024 snapshot

## Evidence

- [results] five categories used to classify producers and how is NP production distributed and shared between them: [evidence span withheld — anchor status: not_found]
- [methods] snapshot taken as primary source: "The snapshot of the PostgreSQL database form the COCONUT release 2024 was taken as the primary source"
- [discussion] provenance not fully provided: "The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter."
- [readme] download options: [evidence span withheld — anchor status: external_doc]
