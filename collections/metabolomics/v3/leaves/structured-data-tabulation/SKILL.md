---
name: structured-data-tabulation
description: Use when an article asserts a count of harmonized or curated source resources
  (e.g. 'over 40 electronic resources') and a structured, count-verified inventory
  of those sources must be extracted from the text.
when_to_use_negative:
- The resource list is already available as a structured table — no text extraction
  is needed
- The task concerns the content of structure-organism pairs (750,000+ pairs) rather
  than enumerating the source resources they were harmonized from
- A current, complete list of LOTUS sources is required — query the hosted data on
  Wikidata or lotus.naturalproducts.net directly, since the paper names only a subset
  of resources
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: lotus-processor
  role: Source repository documenting the harmonization pipeline whose input resources
    are being tabulated
  repo: https://github.com/lotusnprod/lotus-processor
- name: Wikidata
  role: Knowledge base hosting the harmonized LOTUS data, cited as one of the resources
    in the inventory
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:829a14a3d233ff069eb708751365cd79b3b91d0ad043a8405f46317d208c0efe
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - Wikidata
  - lotus-processor
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# structured-data-tabulation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Extracts the names of electronic resources harmonized by LOTUS from the article text and tabulates them into a structured inventory, verifying the count against the paper's claim of harmonization from over 40 electronic resources.

## When to use

Use when an article asserts a count of harmonized or curated source resources (e.g. 'over 40 electronic resources') and a structured, count-verified inventory of those sources must be extracted from the text.

## When NOT to use

- The resource list is already available as a structured table — no text extraction is needed
- The task concerns the content of structure-organism pairs (750,000+ pairs) rather than enumerating the source resources they were harmonized from
- A current, complete list of LOTUS sources is required — query the hosted data on Wikidata or lotus.naturalproducts.net directly, since the paper names only a subset of resources

## Inputs

- Full text of the LOTUS article (DOI 10.7554/elife.70780), especially abstract and results sections
- Source cards with extracted findings naming electronic resources
- Figure 2 (alluvial plot of individual source contributions) as an optional cross-check

## Outputs

- Structured inventory file (TSV/CSV) with one row per electronic resource, each with a name/ID column
- Count verification note comparing extracted rows to the 'over 40 electronic resources' claim

## How to apply

Read the abstract and results of the LOTUS paper (DOI 10.7554/elife.70780) to locate the harmonization claim, then extract every named electronic resource (e.g. COCONUT, KNApSAcK, NAPRALERT, NPAtlas, PubChem, GBIF, DNP, UniProt, Wikidata). Tabulate each into an inventory file with name/ID columns (CSV or TSV). Reconcile the extracted rows against the paper's stated claim of 'over 40 electronic resources' and against the reported finding that LOTUS identified 36 electronic NP resources as valuable input plus DNP and FooDB for comparison/curation; mark unnamed resources explicitly rather than inflating the count. Use the alluvial plot in Figure 2, which illustrates the individual contribution of each source, as a cross-check that the inventory covers the sources contributing structure-organism pairs.

## Related tools

- **lotus-processor** (Source repository documenting the harmonization pipeline whose input resources are being tabulated) — https://github.com/lotusnprod/lotus-processor
- **Wikidata** (Knowledge base hosting the harmonized LOTUS data, cited as one of the resources in the inventory)

## Examples

```
python -c "import pandas as pd; pd.DataFrame(resources, columns=['resource_id','name']).to_csv('lotus_source_inventory.tsv', sep='\t', index=False)"
```

## Evaluation signals

- Inventory row count reconciles with the paper's claim of harmonization from over 40 electronic resources, without padding unnamed entries
- Every row has a non-empty name/ID field; the file parses cleanly as TSV/CSV
- Resources explicitly named in the text (COCONUT, KNApSAcK, NAPRALERT, NPAtlas, PubChem, GBIF, DNP, UniProt, Wikidata) all appear as rows
- Extracted count is consistent with the finding of 36 electronic NP resources as valuable LOTUS input plus DNP and FooDB for comparison
- Inventory agrees with the per-source contributions depicted in the alluvial plot (Figure 2)

## Limitations

- The article explicitly names only a subset of the 40+ resources; the finding reports 36 electronic NP resources plus DNP and FooDB, so a fully named inventory of 40+ rows may not be extractable from the text alone
- DNP and FooDB are noted as not publicly disseminated, limiting verifiability of those inventory entries
- The resource list reflects the 2022 paper; the lotusnprod repositories are archived and development has moved to github.com/lotus-initiative, so the current source list may differ

## Evidence

- [readme] LOTUS is a living database hosted in parallel at Wikidata and LNPN: [evidence span withheld — anchor status: external_doc]
