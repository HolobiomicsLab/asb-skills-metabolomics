---
name: taxonomy-name-cleaning-and-filtering
description: Use when building a taxon-restricted census (e.g. reported compounds
  per family) or a metadata table where species names must be cleaned to up-to-date
  recognized names before counting distinct genera/species.
when_to_use_negative:
- When the input already carries OTU/ASV or resolved taxonomy identifiers rather than
  free-text species names needing synonym reconciliation
- When the goal is chemical novelty prioritization of extracts (Inventa scoring) rather
  than taxon-level counting or name normalization
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3314
tools:
- name: Open Tree of Life
  role: Reference taxonomy used to clean species names to up-to-date recognized names
    before counting or scoring
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: LOTUS
  role: Source database whose reported compound–organism pairs are filtered by family-level
    taxonomy (e.g. Celastraceae) and counted by genus/species
  repo: https://lotus.naturalproducts.net/
- name: pandas
  role: Load and filter the LOTUS dump, deduplicate compounds by canonical structure
    identifiers, and aggregate genus/species counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:27f950200120c2af29e6a3eb53ffbdb76047452770db898c2ef27b5c27d5efeb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LOTUS
  - Open Tree of Life
  - pandas
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# taxonomy-name-cleaning-and-filtering

## Summary

Normalizes organism taxon names (especially species binomials) against currently recognized names using the Open Tree of Life, and filters/facets records by taxonomic rank (family, genus, species) before counting unique natural products, genera, or species in datasets such as LOTUS.

## When to use

Use when building a taxon-restricted census (e.g. reported compounds per family) or a metadata table where species names must be cleaned to up-to-date recognized names before counting distinct genera/species.

## When NOT to use

- When the input already carries OTU/ASV or resolved taxonomy identifiers rather than free-text species names needing synonym reconciliation
- When the goal is chemical novelty prioritization of extracts (Inventa scoring) rather than taxon-level counting or name normalization

## Inputs

- LOTUS database dump with compound–organism pairs and taxonomy columns (lotus.naturalproducts.net)
- GNPS FBMN metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe headers
- Taxonomic reference for name reconciliation (Open Tree of Life)

## Outputs

- Taxon-name-cleaned metadata table or filtered compound–organism pair set
- Counts of unique natural products (canonical SMILES/INCHI), distinct genera, and distinct species per taxonomic group
- Aggregated census file (e.g. lotus_celastraceae_census.csv)

## How to apply

First identify the rank at which filtering is needed (e.g. family-level taxonomy equals Celastraceae in the LOTUS dump, or ATTRIBUTE_Species in a GNPS FBMN metadata table). Clean each species binomial against currently recognized names — the Inventa workflow prescribes using the Open Tree of Life (https://opentree.readthedocs.io) for this reconciliation — and standardize genus/species strings before deduplication. After cleaning, count distinct taxa at each rank (unique natural products via canonical SMILES/INCHI identifiers, distinct genera, distinct species) and validate against reference values (e.g. ~38 genera and 168 species for Celastraceae, where only approximate counts signal acceptable name-cleaning variance). Rationale: uncleaned synonymy would split one taxon into several counts, inflating genus/species figures and mis-scoring literature-based components such as Inventa's Literature Component.

## Related tools

- **Open Tree of Life** (Reference taxonomy used to clean species names to up-to-date recognized names before counting or scoring) — https://opentree.readthedocs.io/en/latest/readme.html
- **LOTUS** (Source database whose reported compound–organism pairs are filtered by family-level taxonomy (e.g. Celastraceae) and counted by genus/species) — https://lotus.naturalproducts.net/
- **pandas** (Load and filter the LOTUS dump, deduplicate compounds by canonical structure identifiers, and aggregate genus/species counts)

## Examples

```
df = pd.read_csv('lotus_dump.tsv', sep='\t'); fam = df[df['organism_family'] == 'Celastraceae']; n_np = fam[['structure_smiles','structure_inchi']].drop_duplicates().shape[0]
```

## Evaluation signals

- Counts must reproduce reported reference values — 4,800 unique NPs, ~38 genera, and 168 species for Celastraceae — with only the 'approximately' qualifier on the genus count tolerating deviation
- Deduplicated compound count is computed on canonical structure identifiers (SMILES/INCHI), not raw entry names
- Number of reported NPClassifier classes (~130, ~20% of Archaeplastida classes) is consistent with the filtered organism subset
- Species names in the output all resolve to recognized names under the Open Tree of Life (no unresolved synonyms in ATTRIBUTE_Species)

## Limitations

- The genus count carries an 'approximately' qualifier, so name resolution may be sensitive to synonym handling and any deviation beyond that qualifier indicates an incorrect filter or deduplication step
- Species cleaning in metadata is a manual prerequisite: users must apply Open Tree of Life reconciliation themselves before Inventa consumes the table

## Evidence

- [readme] Species cleaning via Open Tree of Life: [evidence span withheld — anchor status: external_doc]
- [methods] Open Tree of Life as dataset/tool in methods: "you can use the [Open Tree of Life](https://opentree.readthedocs.io/en/latest/readme.html)"
- [readme] Taxon-based literature scoring depends on cleaned taxonomy: "The Literature Component (LC) is a score based on the number of compounds reported in the literature for the taxon of a given extract."
