---
name: taxonomic-name-resolution-opentree
description: Use when a metadata table's ATTRIBUTE_Species (or genus/family) names
  may be outdated or synonymized and those names will drive taxon-level database queries
  such as LOTUS compound counting for the Inventa LC score.
when_to_use_negative:
- Taxon names are already curated to currently recognized names (e.g., already passed
  through OTL or another synonym resolver) — re-resolution adds no value
- The downstream score does not depend on taxon identity, e.g., computing only the
  Feature Component (FC) or Similarity Component (SC), which are taxon-independent
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_0621
tools:
- name: Open Tree of Life
  role: Taxonomy service used to clean species/genus/family names to up-to-date recognized
    taxa
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: LOTUS
  role: Natural-products database queried with the resolved taxon names to count reported
    compounds per taxonomic level
  repo: https://lotus.naturalproducts.net/
- name: Inventa
  role: 'Consumer of the resolved taxonomy: its LC score subtracts per-level reported-compound
    ratios from 1'
  repo: https://github.com/luigiquiros/inventa
- name: GNPS
  role: Source of the preferred metadata table format whose taxon columns are resolved
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:14d3770ab604f159f5efb36f7b846559ed7c2dab78d393ef0b1fcf5e4e8d6759
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - Inventa
  - LOTUS
  - Open Tree of Life
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# taxonomic-name-resolution-opentree

## Summary

Resolves species, genus, and family names in a GNPS-format metadata table to currently recognized taxa using the Open Tree of Life taxonomy service, so that downstream literature queries (e.g., LOTUS compound counts for Inventa's Literature Component) match the correct taxon.

## When to use

Use when a metadata table's ATTRIBUTE_Species (or genus/family) names may be outdated or synonymized and those names will drive taxon-level database queries such as LOTUS compound counting for the Inventa LC score.

## When NOT to use

- Taxon names are already curated to currently recognized names (e.g., already passed through OTL or another synonym resolver) — re-resolution adds no value
- The downstream score does not depend on taxon identity, e.g., computing only the Feature Component (FC) or Similarity Component (SC), which are taxon-independent

## Inputs

- GNPS-format metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe columns
- Taxon name strings for species, genus, and family of each extract

## Outputs

- Cleaned metadata table with currently recognized taxon names
- Per-taxon reported compound counts from LOTUS feeding into the Literature Component (LC) score

## How to apply

Load the GNPS-format metadata table used for Feature-Based Molecular Networking and inspect the taxon columns (ATTRIBUTE_Species, plus genus and family fields derived from it). Query the Open Tree of Life taxonomy service (https://opentree.readthedocs.io) to clean each name to an up-to-date recognized taxon, replacing synonyms and misspellings before any database lookup. Then use the cleaned species/genus/family names to query LOTUS for reported compound counts per taxonomic level (e.g., Pristimera indica: 2 species-level, 8 genus-level, 6,064 family-level reports). The rationale is that stale names would silently return zero or wrong compound counts and corrupt the LC computation, since LC starts at 1 and subtracts reported-compound ratios against user-defined maxima (sp=20, g=100, f=500) weighted by ws=wg=wf=1.

## Related tools

- **Open Tree of Life** (Taxonomy service used to clean species/genus/family names to up-to-date recognized taxa) — https://opentree.readthedocs.io/en/latest/readme.html
- **LOTUS** (Natural-products database queried with the resolved taxon names to count reported compounds per taxonomic level) — https://lotus.naturalproducts.net/
- **Inventa** (Consumer of the resolved taxonomy: its LC score subtracts per-level reported-compound ratios from 1) — https://github.com/luigiquiros/inventa
- **GNPS** (Source of the preferred metadata table format whose taxon columns are resolved)

## Examples

```
from opentree import OT; taxon = OT.tnrs_match(["Pristimera indica"]); print(taxon.matched_name2)
```

## Evaluation signals

- All resolved species/genus/family names are accepted by the Open Tree of Life taxonomy service without synonym warnings
- LOTUS queries against resolved names return non-zero counts where literature exists (e.g., Pristimera indica: 2 species, 8 genus, 6,064 family reports)
- Computed LC values stay within the defined 0-to-1 range (LC of 1 means no reported compounds for the taxon)
- Spot-check LC: for Pristimera indica with counts 2/8/6064 and maxima 20/100/500 and weights ws=wg=wf=1, LC equals 0.87

## Limitations

- Resolution only fixes taxon nomenclature; it cannot recover compound reports that LOTUS simply does not contain for a poorly studied species (as for Pristimera indica, LC = 0.87 still flags it as poorly studied)
- The LC result also depends on user-defined maxima (max_comp_reported_sp = 20, max_comp_reported_g = 100, max_comp_reported_f = 500) and weights, so resolved names alone do not determine the score

## Evidence

- [readme] ATTRIBUTE_Species names must be resolved: [evidence span withheld — anchor status: not_found]
- [readme] LC subtracts reported-compound fractions from 1: "From this initial value (‘1’), fractions (ratio of reported compounds over the user-defined maximum value of reported compounds) are subtracted."
- [methods] LC is literature-based, not spectral: "The **Literature Component (LC)** is a score based on the number of compounds reported in the literature for the taxon of a given extract."
- [methods] Metadata cleaning precedes taxon queries: [evidence span withheld — anchor status: not_found]
