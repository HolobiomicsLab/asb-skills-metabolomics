---
name: natural-products-database-harmonization
description: Use when merging structure-organism pair records from multiple NP electronic
  resources (e.g. KNApSAcK, NAPRALERT, NPAtlas, PubChem, GBIF) and you need standardized
  chemical, biological and reference levels plus validated retention before open release.
when_to_use_negative:
- The input is a single already-curated NP database that does not need cross-source
  harmonization or validation filtering.
- The goal is structural or spectral searching over the dataset rather than curation
  — Wikidata hosting does not provide these search modes; use the parallel lotus-web
  hosting instead.
- The candidate resources are commercial or not publicly disseminable (e.g. DNP, FooDB),
  which LOTUS used only for comparison/curation, not as open harmonization input.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: lotus-processor
  role: Repository containing the harmonization, processing and validation scripts,
    including filter_dirty.R for automatic filtering of all entries
  repo: https://github.com/lotusnprod/lotus-processor
- name: Wikidata
  role: Host of the harmonized LOTUS data, supporting community curation and elaborate
    SPARQL queries for exploration
  repo: https://www.wikidata.org
- name: lotus-wikidata-interact
  role: 'Programs for data addition to Wikidata: uploadLotus (upload LOTUS data),
    downloadLotus (download/check existing entries), importPublication (create article
    entries from a DOI)'
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
- name: MolVS
  role: 'Python package performing chemical sanitization: standardizing, fragment
    removal and uncharging of structures'
- name: lotus-web
  role: Parallel hosting providing a user-friendly experience (structural search,
    taxonomy-oriented queries, flat table and structure exports) to compensate for
    Wikidata's missing search modes
  repo: https://github.com/lotusnprod/lotus-web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:c7447990f31473eaf896ec306eb0d291be02932c9c6bf7458a3c15a0fa8654ea
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - MolVS
  - Wikidata
  - lotus-processor
  - lotus-web
  - lotus-wikidata-interact
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# natural-products-database-harmonization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Aggregates and harmonizes structure-organism occurrence pairs from 40+ heterogeneous natural product (NP) electronic resources into a standardized, validated open dataset, following the LOTUS three-stage curation pipeline (harmonization, processing, validation) and dissemination via Wikidata. It converts scattered, inconsistent NP records into a FAIR/TRUST-compliant resource released under CC0.

## When to use

Use when merging structure-organism pair records from multiple NP electronic resources (e.g. KNApSAcK, NAPRALERT, NPAtlas, PubChem, GBIF) and you need standardized chemical, biological and reference levels plus validated retention before open release.

## When NOT to use

- The input is a single already-curated NP database that does not need cross-source harmonization or validation filtering.
- The goal is structural or spectral searching over the dataset rather than curation — Wikidata hosting does not provide these search modes; use the parallel lotus-web hosting instead.
- The candidate resources are commercial or not publicly disseminable (e.g. DNP, FooDB), which LOTUS used only for comparison/curation, not as open harmonization input.

## Inputs

- Raw structure-organism pair records from NP electronic resources (e.g. KNApSAcK 50,000+ structures/100,000+ pairs, NAPRALERT 900,000+ records, NPAtlas, COCONUT)
- Chemical structures as SMILES strings
- Reference metadata (DOI, PMID, title, year, journal, first author)
- Taxonomic/organism names and organ names of the producing organism

## Outputs

- Validated, standardized set of referenced structure-organism pairs (750,000+ retained from 2.5M+ initial pairs)
- Structured inventory of harmonized source resources (name/ID per resource, target ≥ 40 resources)
- Open dataset released under CC0, hosted on Wikidata and mirrored on lotus.naturalproducts.net

## How to apply

Collect candidate NP electronic resources and record each as an entry in a source inventory; LOTUS manually assessed source quality and reported over 40 harmonized resources, with 36 electronic NP resources identified as valuable input plus DNP and FooDB reserved for comparison/curation rather than open dissemination. Harmonize each pair at three levels — chemical, biological and reference — then sanitize chemical structures with MolVS in three stages (standardizing, fragment removal, uncharging). Apply validation rules derived from manual curation of 420 referenced pairs: accept references with a DOI or PMID, or titles with restricted Damerau-Levenshtein distance < 10; keep entries with at least two of (year, journal, first author); discard SMILES containing '.' (remaining dimers); keep organism common names only above 3 characters and discard translations where a specific epithet was added or genus/species consistency is below 50%. Run automatic filtering over all entries via filter_dirty.R, re-validate a diverse subset of 100 entries (expect ≥ 97% true positives), and expect to retain roughly 30% of the 2.5M+ initial pairs. Finally, upload retained entries to Wikidata via upsert scripts (lotus-wikidata-interact) and mirror to lotus.naturalproducts.net, which compensates for Wikidata's lack of structural/spectral search modes.

## Related tools

- **lotus-processor** (Repository containing the harmonization, processing and validation scripts, including filter_dirty.R for automatic filtering of all entries) — https://github.com/lotusnprod/lotus-processor
- **Wikidata** (Host of the harmonized LOTUS data, supporting community curation and elaborate SPARQL queries for exploration) — https://www.wikidata.org
- **lotus-wikidata-interact** (Programs for data addition to Wikidata: uploadLotus (upload LOTUS data), downloadLotus (download/check existing entries), importPublication (create article entries from a DOI)) — https://github.com/lotusnprod/lotus-wikidata-interact
- **MolVS** (Python package performing chemical sanitization: standardizing, fragment removal and uncharging of structures)
- **lotus-web** (Parallel hosting providing a user-friendly experience (structural search, taxonomy-oriented queries, flat table and structure exports) to compensate for Wikidata's missing search modes) — https://github.com/lotusnprod/lotus-web

## Examples

```
Rscript filter_dirty.R  # automatic filtering applied to all entries after manual validation of 420 pairs
```

## Evaluation signals

- Source inventory contains at least 40 electronic resources, matching the paper's claim of harmonization of 'over 40 electronic ressources'
- Retention ratio: validated pairs ≈ 30% of initial pairs (750,000+ of 2.5M+), consistent with manual+automatic validation output
- Manual re-validation of a diverse subset of 100 automatically curated entries yields ≥ 97% true positives
- Every retained entry satisfies the validity rule: structure, organism, and reference all described in the reference
- No retained chemical structure contains a '.' in its SMILES (dimer check)

## Limitations

- Chemical sanitization is lossy: correct salts, charged or dimeric molecules in the input suffer an unwanted 'sanitization' step
- The LOTUS processing pipeline is heavy, includes many dependencies and is convoluted
- Name-to-structure and structure-to-name translations rely on predefined rules that do not cover all cases and can commonly lead to incorrect translations
- The pipeline still removes many correct entries and misses or induces some incorrect ones; no exact count is given
- Organ-name removal from producing organisms is needed to avoid wrong translation, but reduces metadata retained

## Evidence

- [readme] lotus-wikidata-interact upload component: [evidence span withheld — anchor status: external_doc]
