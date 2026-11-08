---
name: structure-organism-pair-export-formatting
description: Use when a validated structure-organism-reference table (e.g. LOTUS table.tsv.gz
  from Zenodo 10.5281/zenodo.5794106) must be converted into Wikidata upsert records
  for batch upload.
when_to_use_negative:
- Input is a raw, uncurated pair set — the export consumes the post-harmonization,
  post-validation table retained after manual (420 pairs) and automatic filtering,
  not the 2.5M+ initial pairs
- The goal is structural or spectral search over the exported data — Wikidata does
  not support these query modes, which is why LOTUS is dual-hosted at lotus.naturalproducts.net
- Only a handful of records need adding — the article notes users can manually add
  data directly in Wikidata instead of running the batch upsert programs
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: NPimporterBot
  role: Wikidata bot whose programs perform the batch upsert of exported structure-organism
    pair records; its activity overview is used to validate the export
- name: SPARQL
  role: Queries Wikidata to check whether compounds, organisms and references already
    exist before upserting, and to reconcile sampled export records against live entries
- name: lotus-wikidata-interact (npwdimporter)
  role: Repository holding uploadLotus (uploads LOTUS data to Wikidata) and downloadLotus
    (checks what already exists on Wikidata), plus importPublication for creating
    article entries from a DOI
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:f125792558a1efdeb9c5c99dd0f120091f5ddb1483ba56cc4a03a1c0b1d72688
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-wikidata-interact
  tools:
  - NPimporterBot
  - SPARQL
  - lotus-wikidata-interact (npwdimporter)
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# structure-organism-pair-export-formatting

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstructs the LOTUS Wikidata export module that converts the curated table.tsv.gz of validated referenced structure-organism pairs into Wikidata-format upsert records, using the lotus-wikidata-interact (npwdimporter) programs that group references, organisms and compounds and check existing Wikidata entities before insertion or update.

## When to use

Use when a validated structure-organism-reference table (e.g. LOTUS table.tsv.gz from Zenodo 10.5281/zenodo.5794106) must be converted into Wikidata upsert records for batch upload.

## When NOT to use

- Input is a raw, uncurated pair set — the export consumes the post-harmonization, post-validation table retained after manual (420 pairs) and automatic filtering, not the 2.5M+ initial pairs
- The goal is structural or spectral search over the exported data — Wikidata does not support these query modes, which is why LOTUS is dual-hosted at lotus.naturalproducts.net
- Only a handful of records need adding — the article notes users can manually add data directly in Wikidata instead of running the batch upsert programs

## Inputs

- curated LOTUS table.tsv.gz of validated referenced structure-organism pairs (Zenodo 10.5281/zenodo.5794106)
- Wikidata property mappings for chemical structure, biological organism and reference fields
- existing Wikidata state (via SPARQL endpoint or direct connection) for duplicate/upsert detection

## Outputs

- wikidata_structure_organism_pairs.csv containing Wikidata-format upsert records for each referenced structure-organism pair
- grouped entity records (compounds, organisms, references) ready for batch upload via npwdimporter
- SPARQL reconciliation results for sampled exported pairs

## How to apply

Load the curated LOTUS table.tsv.gz from the Zenodo deposit with pandas and map each row's chemical structure, biological organism and reference fields onto the Wikidata properties used by LOTUS, keeping the referenced structure-organism pair as the unit of export. Group records so that references, organisms and compounds are handled together, mirroring how the LOTUS Wikidata importer treats them as linked entities rather than isolated items. Before generating records, check whether each entity already exists in Wikidata using SPARQL queries or a direct connection, as the downloadLotus component is used by uploadLotus for this pre-check. Emit insert-or-update (upsert) records for every referenced pair via the batch upload programs in the lotus-wikidata-interact repository (the npwdimporter/upsert scripts), writing the result to wikidata_structure_organism_pairs.csv. Validate the export by confirming the file exists and parses, and by reconciling sampled pair records against live Wikidata entries via SPARQL, as surfaced by the NPimporterBot activity overview.

## Related tools

- **NPimporterBot** (Wikidata bot whose programs perform the batch upsert of exported structure-organism pair records; its activity overview is used to validate the export)
- **SPARQL** (Queries Wikidata to check whether compounds, organisms and references already exist before upserting, and to reconcile sampled export records against live entries)
- **lotus-wikidata-interact (npwdimporter)** (Repository holding uploadLotus (uploads LOTUS data to Wikidata) and downloadLotus (checks what already exists on Wikidata), plus importPublication for creating article entries from a DOI) — https://github.com/lotusnprod/lotus-wikidata-interact

## Examples

```
import pandas as pd; df = pd.read_csv('table.tsv.gz', sep='\t')
```

## Evaluation signals

- wikidata_structure_organism_pairs.csv exists and is parseable
- Sampled pair records can be reconciled against existing Wikidata entries via SPARQL queries
- Every exported record carries all three curated fields: chemical structure, biological organism and reference
- Re-running the export over the same table.tsv.gz yields upsert (update) rather than duplicate entity creation for already-published pairs
- NPimporterBot activity overview reflects uploads corresponding to the exported record set (750,000+ validated pairs)

## Limitations

- The upstream LOTUS processing pipeline is heavy, includes many dependencies and is convoluted; it still removes correct entries and misses or induces incorrect ones
- Wikidata lacks structural and spectral search modes, limiting what can be done with the exported data in Wikidata alone
- The lotus-wikidata-interact repository is archived and development has moved to github.com/lotus-initiative

## Evidence

- [readme] [uploadLotus](uploadLotus/README.md) : Used to upload LOTUS data to Wikidata: [evidence span withheld — anchor status: external_doc]
- [readme] [downloadLotus](downloadLotus/README.md) : Used to download LOTUS data from Wikidata: [evidence span withheld — anchor status: external_doc]
