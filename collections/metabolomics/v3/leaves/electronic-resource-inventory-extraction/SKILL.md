---
name: electronic-resource-inventory-extraction
description: Use when an article claims harmonization/curation from a named number
  of electronic resources (e.g. 'over 40 electronic resources') and a verifiable,
  structured source inventory must be extracted from the text.
when_to_use_negative:
- The input is a structured database dump or API listing where resources can be enumerated
  programmatically instead of read from prose
- The goal is harmonizing/curating the actual structure-organism pairs rather than
  inventorying the sources that contributed them
- The article names no specific source resources, so only a count claim with no extractable
  names exists
edam_operation: http://edamontology.org/operation_2422
tools:
- name: lotus-processor
  role: Repository whose scripts document the harmonization pipeline over the input
    sources; its wiki can be cross-checked to confirm the enumerated resources
  repo: https://github.com/lotusnprod
- name: Wikidata
  role: Host knowledge base whose SPARQL endpoint can be queried to confirm which
    source resources are represented as provenance for LOTUS entries
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:f874b0b5b39b5d7e0a1ec5a0ae0ffabd891e31f1eaa8841a4ca278077754e871
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Wikidata
  - lotus-processor
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# electronic-resource-inventory-extraction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Extracts and tabulates the electronic resources that a harmonization effort (here, the LOTUS natural products knowledge base) declares as its data inputs, into a structured inventory with name/ID columns that can be counted against the article's stated claim.

## When to use

Use when an article claims harmonization/curation from a named number of electronic resources (e.g. 'over 40 electronic resources') and a verifiable, structured source inventory must be extracted from the text.

## When NOT to use

- The input is a structured database dump or API listing where resources can be enumerated programmatically instead of read from prose
- The goal is harmonizing/curating the actual structure-organism pairs rather than inventorying the sources that contributed them
- The article names no specific source resources, so only a count claim with no extractable names exists

## Inputs

- Full-text article (e.g. DOI 10.7554/elife.70780) with abstract, methods and results sections
- Named source-count claim from the article (e.g. 'over 40 electronic resources')
- Optional: lotus-processor repository/wiki listing upstream data sources

## Outputs

- Structured inventory table (CSV/TSV) with one row per electronic resource and a name/ID column
- Count verification report comparing extracted rows against the article's claimed resource count
- Flagged list of comparison/curation-only resources (e.g. DNP, FooDB) distinct from harmonized inputs

## How to apply

Locate the abstract or methods passage stating the source-count claim (LOTUS: 'over 40 electronic ressources' harmonized across chemical, biological and reference levels). Scan the full text for every named electronic resource — in LOTUS these include KNApSAcK (50,000+ structures, 100,000+ structure-organism pairs), NAPRALERT (900,000+ records), NPAtlas, COCONUT, PubChem (>100 M entries), GBIF (>1900 M entries), UniProt, DNP and FooDB. Distinguish resources used as harmonized LOTUS input from comparison/curation-only sources: the article identifies 36 electronic NP resources as valuable LOTUS input, while DNP and FooDB are commercial or non-disseminated resources used for comparison/curation. Tabulate each resource into an inventory table with a name/ID column, then verify the row count against the paper's claim; a shortfall (36 found vs 40+ claimed) is itself a reportable finding, since some sources may be aggregated or unnamed.

## Related tools

- **lotus-processor** (Repository whose scripts document the harmonization pipeline over the input sources; its wiki can be cross-checked to confirm the enumerated resources) — https://github.com/lotusnprod
- **Wikidata** (Host knowledge base whose SPARQL endpoint can be queried to confirm which source resources are represented as provenance for LOTUS entries)

## Evaluation signals

- Inventory row count is at least 40, or the discrepancy with the article's claim (36 explicitly identified as valuable LOTUS input) is explicitly documented
- Every row has a non-empty name/ID and is traceable to a verbatim quote in the article text
- DNP and FooDB are present but flagged as comparison/curation-only (not publicly disseminated), not as harmonized LOTUS inputs
- Named resources match the article's stated scales: KNApSAcK 50,000+ structures, NAPRALERT 900,000+ records, PubChem >100 M entries, GBIF >1900 M entries
- No duplicate resource entries under alternate spellings (e.g. 'DNP' vs 'Dictionary of Natural Products' collapsed to one row)

## Limitations

- The abstract's 'over 40 electronic resources' claim does not fully reconcile with the 36 electronic NP resources the article identifies as valuable LOTUS input, so the inventory may undershoot the claimed count
- Resources used only for comparison or curation (DNP, FooDB) are not publicly disseminated, so their role cannot be verified against external data
- Only resources explicitly named in the text can be extracted; aggregated or unnamed feeds remain invisible to this method

## Evidence

- [abstract] Named resource with scale metadata for the inventory: [evidence span withheld — anchor status: not_found]
