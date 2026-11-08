---
name: chemical-nomenclature-synonym-mapping
description: Reference for constructing a synonym table for a chemical taxonomy, especially
  when resolving competing class names and cross-ontology mappings (ChEBI, LIPID MAPS,
  MeSH) for classification text.
when_to_use_negative:
- Your ontology is already fully harmonized with the external target vocabulary and
  every term already carries scoped synonyms — the skill adds value only when alternate
  or cross-ontology names must be resolved and consolidated.
- You need to classify individual structures rather than harmonize vocabulary — that
  is ClassyFire's rule-based structural classification task, not synonym mapping.
- Names refer to individual compounds rather than chemical classes/categories — synonym
  resolution at the compound level (e.g. compound names → InChI) is a different task.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3345
tools:
- name: ClassyFire
  role: Web server and program whose underlying taxonomy (ChemOnt) is the target vocabulary
    being synonym-mapped; the OBO export is obtained via the ClassyFire website
  repo: http://classyfire.wishartlab.com
- name: OBO-Edit
  role: Used to generate the OBO-format export of the ontology that is parsed for
    terms, definitions, and synonym stanzas
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:bce7ab4b3c471e557f844255c166d7d0fcf4e010aaa4564bbb0edbde1eaae7c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - OBO-Edit
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# chemical-nomenclature-synonym-mapping

## Summary

Build a comprehensive synonym dictionary that maps alternate chemical class names (including IUPAC, MeSH, and other ontology variants) onto preferred ontology terms, as done for the 4825-category ChemOnt taxonomy underlying ClassyFire.

## When to use

Reference for constructing a synonym table for a chemical taxonomy, especially when resolving competing class names and cross-ontology mappings (ChEBI, LIPID MAPS, MeSH) for classification text.

## When NOT to use

- Your ontology is already fully harmonized with the external target vocabulary and every term already carries scoped synonyms — the skill adds value only when alternate or cross-ontology names must be resolved and consolidated.
- You need to classify individual structures rather than harmonize vocabulary — that is ClassyFire's rule-based structural classification task, not synonym mapping.
- Names refer to individual compounds rather than chemical classes/categories — synonym resolution at the compound level (e.g. compound names → InChI) is a different task.

## Inputs

- Chemical ontology in OBO format (e.g. ChemOnt, downloadable from the ClassyFire website)
- Usage-frequency statistics for candidate class names (e.g. Google hit counts)
- Cross-ontology mapping candidates (ChEBI, LIPID MAPS, MeSH terms)

## Outputs

- Chemical Classification Dictionary: preferred term → synonym table (~9,012 English synonyms for ChemOnt's 4,825 categories)
- Scope-tagged ontology cross-mappings (exact/narrow/broad/related) to ChEBI, LIPID MAPS, and MeSH
- 20–50 word textual definitions per category for generating compound descriptions

## How to apply

Starting from a chemical ontology exported in OBO format (e.g. ChemOnt from the ClassyFire website), extract term IDs, preferred names, and text definitions for each category, then compile synonym stanzas into a mapping table keyed by preferred term. When multiple candidate names describe the same category, choose the term with the highest usage frequency — the paper's example picked 'Imidazolines' (229,000 Google hits) over 'Dihydroimidazoles' (4,590) — giving precedence to IUPAC-recommended terms and expert-panel choices, while retaining the rejected terms as synonyms. For cross-ontology alignment, tag each synonym with its mapping scope (exact, narrow, broad, or related) when linking to ChEBI, LIPID MAPS, or MeSH, e.g. ChEBI's 'carboxylic acid' as a narrow synonym of ChemOnt's 'carboxylic acids'. The result should cover every category in the taxonomy; for ChemOnt this yielded ~9,012 English synonym entries across 4,825 categories (4,146 organic, 678 inorganic, plus the root 'Chemical entities'), each also carrying a 20–50 word definition usable for automated textual description of compounds.

## Related tools

- **ClassyFire** (Web server and program whose underlying taxonomy (ChemOnt) is the target vocabulary being synonym-mapped; the OBO export is obtained via the ClassyFire website) — http://classyfire.wishartlab.com
- **OBO-Edit** (Used to generate the OBO-format export of the ontology that is parsed for terms, definitions, and synonym stanzas)

## Evaluation signals

- Coverage check: the synonym table must contain an entry for every category in the taxonomy (4,825 for ChemOnt) with ~9,012 English synonym entries in total.
- Spot-check documented resolution pairs: 'Imidazolines' retained as preferred over 'Dihydroimidazoles' based on 229,000 vs 4,590 Google hits.
- Verify each synonym carries a scope tag (exact, narrow, broad, or related) where it is mapped to ChEBI, LIPID MAPS, or MeSH.
- Confirm the preferred-name choices favor IUPAC terms and expert-panel recommendations over less-frequent synonyms.
- Validate that each dictionary entry carries a 20–50 word definition suitable for generating compound descriptions.

## Limitations

- Usage-frequency resolution (e.g. Google hit counts) is a heuristic; competing terms with comparable usage may need expert-panel or IUPAC-preference adjudication.
- Mappings to external ontologies such as MeSH are only partially complete — the paper notes ClassyFire was 'manually mapped, although only partially, to the MeSH thesaurus'.
- Category descriptions define preferred compound memberships, and compounds that undergo reactions (e.g. loss of oxygen, substitutions) may fall outside the constraints of their expected category description, complicating synonym-driven annotation.

## Evidence

- [abstract] Chemical Classification Dictionary with thousands of 20–50 word definitions: "we describe a comprehensive, flexible, computable, chemical taxonomy along with a fully annotated chemical ontology (ChemOnt) and a Chemical Classification Dictionary"
- [methods] OBO file generation and download path: "The resulting OBO file was generated with OBO-Edit [34], and can be downloaded from the ClassyFire website."
- [methods] Partially incomplete MeSH mapping: "ClassyFire has also been manually mapped, although only partially, to the MeSH thesaurus, which is used in the PubChem database."
