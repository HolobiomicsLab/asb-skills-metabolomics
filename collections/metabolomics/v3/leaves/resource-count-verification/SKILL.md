---
name: resource-count-verification
description: Use when a paper claims a dataset was harmonized from a numeric minimum
  of source resources (e.g. 'over 40 electronic resources') and you must audit that
  claim by enumerating the resources into an inventory.
when_to_use_negative:
- When the paper does not state a numeric threshold for resource count — extraction
  would produce an unverifiable list
- When the goal is to use the resources themselves rather than audit the count — this
  skill inventories names only, it does not harvest data from them
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3077
tools:
- name: lotus-processor
  role: Repository hosting the LOTUS harmonization pipeline; the skill audits the
    resource inventory that this pipeline consumed
  repo: https://github.com/lotusnprod/lotus-processor
- name: Wikidata
  role: Host knowledge base where harmonized structure-organism pairs and resource
    provenance are mirrored
  repo: https://www.wikidata.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:efd2d325031cbc1ca5d99211f917a1a7c429cd1087b145b430d1bf9b553acd8e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - Wikidata
  - lotus-processor
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# resource-count-verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Extracts the named electronic natural-product resources that LOTUS harmonized from the article text and tabulates them into a structured inventory, verifying that the count meets the paper's stated threshold of over 40 electronic resources.

## When to use

Use when a paper claims a dataset was harmonized from a numeric minimum of source resources (e.g. 'over 40 electronic resources') and you must audit that claim by enumerating the resources into an inventory.

## When NOT to use

- When the paper does not state a numeric threshold for resource count — extraction would produce an unverifiable list
- When the goal is to use the resources themselves rather than audit the count — this skill inventories names only, it does not harvest data from them

## Inputs

- Scientific article full text (results and abstract sections) that claims a numeric count of harmonized electronic resources

## Outputs

- Structured inventory file (CSV/TSV) with one row per extracted electronic resource, including name/ID columns
- Resource count verification report confirming the extracted count meets the paper's stated threshold (>= 40)

## How to apply

Read the methods/results section of the LOTUS paper to locate the passage stating that data was curated and harmonized from over 40 electronic resources. Extract the names of each electronic resource mentioned as harmonized by LOTUS, then tabulate them into a structured inventory file with a name/ID column. Verify the extracted count is at least 40, matching the paper's claim; the article notes LOTUS identified thirty-six electronic NP resources as valuable input plus DNP and FooDB for comparison/curation, so tally both sets. The output is a structured inventory file (e.g. CSV/TSV) with one row per resource.

## Related tools

- **lotus-processor** (Repository hosting the LOTUS harmonization pipeline; the skill audits the resource inventory that this pipeline consumed) — https://github.com/lotusnprod/lotus-processor
- **Wikidata** (Host knowledge base where harmonized structure-organism pairs and resource provenance are mirrored) — https://www.wikidata.org

## Evaluation signals

- Inventory contains >= 40 rows when counting the thirty-six electronic NP resources plus DNP and FooDB comparison resources
- Every inventory row has a non-empty name/ID that matches a resource mentioned in the article text
- No duplicate resource entries in the inventory
- The reported count in the inventory matches the paper's stated 'over 40 electronic resources' claim
- Extraction is traceable to a verbatim passage in the article's abstract or results section

## Limitations

- The article does not enumerate all 40+ resources in a single list; some are mentioned only in the abstract or scattered across sections, so extraction may miss implicitly-harmonized resources
- The count 'over 40' is not a precise number, so verification is a lower-bound (>= 40) rather than an exact match
- Resources listed for comparison/curation (DNP, FooDB) are not publicly disseminated by LOTUS, so their role differs from the thirty-six integrated NP resources and may confuse a naive count

## Evidence

- [methods] Enumeration of the 36 NP resources plus comparison resources: [evidence span withheld — anchor status: not_found]
- [results] Additional resource mentions in the results section: [evidence span withheld — anchor status: not_found]
