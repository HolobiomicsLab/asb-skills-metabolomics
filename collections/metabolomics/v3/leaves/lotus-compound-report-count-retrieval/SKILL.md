---
name: lotus-compound-report-count-retrieval
description: Use when computing Inventa's Literature Component (LC) requires per-taxon
  compound report counts from LOTUS at the species, genus, and family levels for each
  sample in a GNPS-format metadata table.
when_to_use_negative:
- The score components being computed require spectral data (e.g. Feature Component
  or Similarity Component via MEMO) — LC is independent of the spectral data
- Taxon names in the metadata are not cleaned/resolved (e.g. synonyms not normalized
  via Open Tree of Life), so LOTUS lookups would silently miss reports
- The target is chemical-class novelty (Class Component via CANOPUS) rather than literature
  coverage of reported compounds
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: LOTUS
  role: Source database queried for the number of compounds reported per species,
    genus, and family
  repo: https://lotus.naturalproducts.net/
- name: Open Tree of Life
  role: Taxonomy service used to clean species names to currently recognized names
    before LOTUS queries
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: Inventa
  role: Consumes the retrieved report counts to compute the Literature Component (LC)
    score within the priority rank
  repo: https://github.com/luigiquiros/inventa
- name: GNPS
  role: Provides the standard-format metadata table carrying the sample taxon fields
    (ATTRIBUTE_Species)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:17e2fea4e2bce52a3778c6b89df349a812675e7f7fb3c0e0f0a310972f95ff74
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - Inventa
  - LOTUS
  - Open Tree of Life
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lotus-compound-report-count-retrieval

## Summary

Retrieves the number of natural compounds reported in the LOTUS database for each extract's species, genus, and family, providing the literature-report counts that feed Inventa's Literature Component (LC) score.

## When to use

Use when computing Inventa's Literature Component (LC) requires per-taxon compound report counts from LOTUS at the species, genus, and family levels for each sample in a GNPS-format metadata table.

## When NOT to use

- The score components being computed require spectral data (e.g. Feature Component or Similarity Component via MEMO) — LC is independent of the spectral data
- Taxon names in the metadata are not cleaned/resolved (e.g. synonyms not normalized via Open Tree of Life), so LOTUS lookups would silently miss reports
- The target is chemical-class novelty (Class Component via CANOPUS) rather than literature coverage of reported compounds

## Inputs

- GNPS-format metadata table with ATTRIBUTE_Species (and organ) columns, cleaned to currently recognized taxon names
- Taxon names resolved against the Open Tree of Life taxonomy service
- User-defined parameters: max_comp_reported_sp = 20, max_comp_reported_g = 100, max_comp_reported_f = 500, and weights ws, wg, wf (all = 1)

## Outputs

- Per-extract compound report counts at species, genus, and family levels (e.g. Pristimera indica: 2, 8, 6,064)
- Per-sample Literature Component (LC) scores written to literature_component_scores.tsv

## How to apply

First clean the species/genus/family names in the GNPS-format metadata table to currently recognized taxa using the Open Tree of Life taxonomy service, since LOTUS lookups require valid names. Then query the LOTUS natural-products database (lotus.naturalproducts.net) for the number of compounds reported at each of the three taxonomic levels for every extract — for the Celastraceae example, Pristimera indica had 2 species-level, 8 genus-level, and 6,064 family-level reports. These counts feed the LC computation, which starts at 1 and subtracts, per level, the fraction of reported compounds over the user-defined maxima (max_comp_reported_sp = 20, max_comp_reported_g = 100, max_comp_reported_f = 500), each modulated by weights ws = wg = wf = 1. The rationale is that a taxon with many literature reports is chemically well studied and thus less likely to yield novel compounds. Assemble the per-sample LC values together with their species/genus/family report counts and write them to literature_component_scores.tsv.

## Related tools

- **LOTUS** (Source database queried for the number of compounds reported per species, genus, and family) — https://lotus.naturalproducts.net/
- **Open Tree of Life** (Taxonomy service used to clean species names to currently recognized names before LOTUS queries) — https://opentree.readthedocs.io/en/latest/readme.html
- **Inventa** (Consumes the retrieved report counts to compute the Literature Component (LC) score within the priority rank) — https://github.com/luigiquiros/inventa
- **GNPS** (Provides the standard-format metadata table carrying the sample taxon fields (ATTRIBUTE_Species))

## Evaluation signals

- For the Celastraceae validation case, Pristimera indica (roots extract) must have report counts 2 (species), 8 (genus), and 6,064 (family) and a computed LC of 0.87
- All computed LC values fall within the component's defined 0-to-1 ratio range
- Counts are non-decreasing across taxonomic levels (family >= genus >= species reports)
- Output file literature_component_scores.tsv contains one LC value plus its three report counts per sample
- A taxon with zero reported compounds at all levels yields LC = 1 (no literature coverage)

## Limitations

- LC is computed from literature coverage only and is independent of the MS spectral data, so it cannot capture novelty of unreported but present compounds
- Species, genus, and family names must be cleaned to currently recognized taxa (via Open Tree of Life) before querying LOTUS, or counts will be wrong
- The LC result depends on user-defined maxima (max_comp_reported_sp/g/f = 20/100/500) and weights (ws, wg, wf), which are tunable and affect the score scale

## Evidence

- [methods] LC definition: "The **Literature Component (LC)** is a score based on the number of compounds reported in the literature for the taxon of a given extract."
- [readme] LC computation from reported-compound fractions: "From this initial value ('1'), fractions (ratio of reported compounds over the user-defined maximum value of reported compounds) are subtracted."
- [readme] Taxon name cleaning requirement: [evidence span withheld — anchor status: external_doc]
- [readme] LC independence from spectral data: [evidence span withheld — anchor status: not_found]
