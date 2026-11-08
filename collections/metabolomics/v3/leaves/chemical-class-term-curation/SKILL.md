---
name: chemical-class-term-curation
description: Reference for constructing or auditing a chemical classification dictionary
  — when you must choose preferred labels and synonyms for chemical classes (e.g.
  resolving 'Imidazolines' vs 'Dihydroimidazoles') rather than classify individual
  structures.
when_to_use_negative:
- Input is a set of molecular structures needing class assignment — use ClassyFire's
  rule-based classification instead; this skill only curates category terminology.
- The taxonomy already carries stable preferred labels and synonyms and you are not
  auditing term choices or adding mappings.
edam_operation: http://edamontology.org/operation_0560
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: OBO-Edit
  role: Used to generate the resulting OBO file that stores the curated ChemOnt terms,
    synonyms, and definitions
- name: ClassyFire
  role: Source of the ChemOnt taxonomy whose category labels, definitions, and synonyms
    are curated
  repo: https://classyfire.wishartlab.com
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:83ba273bffc2906fb728bd514fbac1c41d269fa3ccf23c0aa5bec3c7c7efeea6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  edam_operation: http://edamontology.org/operation_0560
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# chemical-class-term-curation

## Summary

Curates the terminology layer of a computable chemical taxonomy: for each of the 4825 ChemOnt categories, it selects a preferred English term (favoured by IUPAC naming, expert-panel recommendation, and recorded usage frequency) and attaches the losing candidate names as synonyms, yielding roughly 9012 English synonyms plus a text definition per node.

## When to use

Reference for constructing or auditing a chemical classification dictionary — when you must choose preferred labels and synonyms for chemical classes (e.g. resolving 'Imidazolines' vs 'Dihydroimidazoles') rather than classify individual structures.

## When NOT to use

- Input is a set of molecular structures needing class assignment — use ClassyFire's rule-based classification instead; this skill only curates category terminology.
- The taxonomy already carries stable preferred labels and synonyms and you are not auditing term choices or adding mappings.

## Inputs

- ChemOnt taxonomy in OBO format (4825 categories: 4146 organic, 678 inorganic, plus the root 'Chemical entities')
- Term-usage statistics (e.g. Google hit counts) for competing class names
- External mapping sources: ChEBI, LIPID MAPS, MeSH
- Expert-panel naming recommendations

## Outputs

- Chemical Classification Dictionary: ~9012 English synonym entries across 4825 categories
- Preferred-label assignment per category with recorded alternate names as synonyms
- Cross-vocabulary mapping table with exact/narrow/broad/related scope tags
- Per-category text definitions of 20–50 words suitable for generating compound descriptions

## How to apply

Download the ChemOnt taxonomy in OBO format from the ClassyFire website and parse each of the 4825 category stanzas (4146 organic, 678 inorganic, plus the root 'Chemical entities'). For each category, gather the competing candidate terms; give precedence to IUPAC terms and expert-panel recommendations, and otherwise resolve ties by usage frequency (e.g. 'Imidazolines' with 229,000 Google hits beats 'Dihydroimidazoles' with 4590). Record the selected term as the preferred name and the rejected names as synonyms, so the losing terms remain discoverable. Additionally map terms to external vocabularies (ChEBI, LIPID MAPS, MeSH), tagging each mapping as exact, narrow, broad, or related (e.g. ChEBI 'carboxylic acid' as a narrow synonym of ChemOnt 'carboxylic acids'). Author a 20–50 word text definition per category so the dictionary can generate English descriptions of individual compounds.

## Related tools

- **OBO-Edit** (Used to generate the resulting OBO file that stores the curated ChemOnt terms, synonyms, and definitions)
- **ClassyFire** (Source of the ChemOnt taxonomy whose category labels, definitions, and synonyms are curated) — https://classyfire.wishartlab.com

## Evaluation signals

- The synonym table covers all 4825 categories with roughly 9012 English synonym entries.
- Every category has a preferred term and a 20–50 word text definition.
- The OBO file parses without error and spot-checks such as the Imidazolines/Dihydroimidazoles pair resolve to the paper's chosen term.
- Cross-vocabulary mappings carry the correct scope tag (e.g. ChEBI 'carboxylic acid' is tagged narrow relative to ChemOnt 'carboxylic acids').

## Limitations

- Conflicting class-name precedents must be settled by a panel; automated frequency counts alone may not reflect expert conventions.
- Terms reserved for a small number of compounds can sit at Class/SubClass level while generic terms belong at SuperClass level, so level conventions must be verified term by term.

## Evidence

- [methods] Term precedence by IUPAC and usage frequency: [evidence span withheld — anchor status: not_found]
- [results] Dictionary scale and role: [evidence span withheld — anchor status: not_found]
- [methods] OBO output format: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
