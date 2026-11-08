---
name: literature-component-subtractive-scoring
description: Use when prioritizing natural extracts for chemical novelty and you need
  a literature-coverage score per taxon, computed independently of spectral data,
  from reported compound counts in LOTUS for species, genus, and family.
when_to_use_negative:
- You need a score that depends on the extract's spectral features or annotations
  — LC is explicitly independent of the spectral data and relies only on taxon literature
  counts.
- The taxon names in the metadata are unvalidated synonyms — clean them with Open
  Tree of Life first, otherwise LOTUS lookups will miss reports and inflate LC.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: Inventa
  role: Computes the LC component by subtracting ratios of reported compounds over
    user-defined maxima from 1
  repo: https://github.com/luigiquiros/inventa.git
- name: LOTUS
  role: Source of the number of compounds reported in the literature per taxon (species,
    genus, family)
- name: Open Tree of Life
  role: Taxonomy service used to clean species/genus/family names to currently recognized
    taxa before LOTUS queries
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: GNPS
  role: Provides the standard-format metadata table whose taxon fields feed the LC
    computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:fee4ac5be35aca9e4b402cfc4a021617c5c6e3a7ed0d1646ce4cadae92f1e79d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa.git
  tools:
  - GNPS
  - Inventa
  - LOTUS
  - Open Tree of Life
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# Literature Component (LC) subtractive scoring for taxon literature coverage

## Summary

Computes Inventa's Literature Component (LC), a score quantifying how much literature coverage exists for an extract's taxon by starting at 1 and subtracting fractions of LOTUS-reported compounds at species, genus, and family levels over user-defined maxima. Low LC values flag poorly studied taxa as candidates for novel-compound discovery.

## When to use

Use when prioritizing natural extracts for chemical novelty and you need a literature-coverage score per taxon, computed independently of spectral data, from reported compound counts in LOTUS for species, genus, and family.

## When NOT to use

- You need a score that depends on the extract's spectral features or annotations — LC is explicitly independent of the spectral data and relies only on taxon literature counts.
- The taxon names in the metadata are unvalidated synonyms — clean them with Open Tree of Life first, otherwise LOTUS lookups will miss reports and inflate LC.

## Inputs

- GNPS-format metadata table with cleaned ATTRIBUTE_Species / genus / family columns
- LOTUS compound-report counts per taxon (species, genus, family levels)
- User-defined parameters: max_comp_reported_sp, max_comp_reported_g, max_comp_reported_f and taxonomic-level weights ws, wg, wf

## Outputs

- literature_component_scores.tsv with per-sample LC values and species/genus/family report counts
- Per-extract LC score in the 0-to-1 range feeding the Inventa priority rank

## How to apply

First clean the species/genus/family names in the GNPS-format metadata table (ATTRIBUTE_Species, etc.) to currently recognized taxa using the Open Tree of Life taxonomy service. Query the LOTUS natural-products database for the number of compounds reported per taxon for each extract (e.g., Pristimera indica: 2 at species level, 8 at genus, 6,064 at family). Compute LC by starting from 1 and subtracting, per taxonomic level, the ratio of reported compounds over the user-defined maximum (max_comp_reported_sp = 20, max_comp_reported_g = 100, max_comp_reported_f = 500), each modulated by taxonomic-level weights (ws = wg = wf = 1). An LC of 1 means no reported compounds for the taxon; for Pristimera indica the components give LC = 0.87, indicating a poorly studied species. Assemble per-sample LC values together with their species/genus/family report counts and write them to literature_component_scores.tsv for use in the Inventa priority rank.

## Related tools

- **Inventa** (Computes the LC component by subtracting ratios of reported compounds over user-defined maxima from 1) — https://github.com/luigiquiros/inventa.git
- **LOTUS** (Source of the number of compounds reported in the literature per taxon (species, genus, family))
- **Open Tree of Life** (Taxonomy service used to clean species/genus/family names to currently recognized taxa before LOTUS queries) — https://opentree.readthedocs.io/en/latest/readme.html
- **GNPS** (Provides the standard-format metadata table whose taxon fields feed the LC computation)

## Examples

```
LC = 1 - (ws*(2/20) + wg*(8/100) + wf*(6064/500 capped))  # per Inventa: start at 1, subtract per-level reported/max fractions
```

## Evaluation signals

- Recompute LC for Pristimera indica roots extract from report counts 2/8/6,064 and confirm it equals 0.87.
- Verify all LC values fall within the component's defined 0-to-1 ratio range.
- Check LC = 1 exactly for taxa with zero reported compounds at all three levels.
- Confirm the output literature_component_scores.tsv contains LC alongside the species/genus/family report counts for every sample.
- Confirm taxon names were resolved against Open Tree of Life before LOTUS counting (no unmatched/synonym taxa).

## Limitations

- LC depends entirely on the completeness of LOTUS reports; an LC of 1 only means no reported compounds for the considered taxon in that database.
- Requires user-defined maxima and weights (max_comp_reported_sp/g/f, ws/wg/wf); different choices change the score and are subjective preferences.
- Taxon names must be cleaned to currently recognized names first; stale synonyms will undercount reports.

## Evidence

- [readme] LC definition: "The **Literature Component (LC)** is a score based on the number of compounds reported in the literature for the taxon of a given extract. It is independent of the spectral data."
- [readme] LC subtraction mechanics: "From this initial value (‘1’), fractions (ratio of reported compounds over the user-defined maximum value of reported compounds) are subtracted."
- [readme] Taxon cleaning requirement: [evidence span withheld — anchor status: external_doc]
- [abstract] Pristimera indica highlighted: "The Pristimera indica (Willd.) A.C.Sm roots extract was highlighted as a promising source of potentially novel compounds"
