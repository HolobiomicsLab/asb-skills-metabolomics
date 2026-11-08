---
name: wikidata-property-mapping
description: Use when a curated, validated TSV of structure-organism-reference pairs
  (e.g. LOTUS table.tsv.gz from Zenodo 10.5281/zenodo.5794106) must be converted into
  Wikidata-formatted upsert records for batch import.
when_to_use_negative:
- Data has not yet passed LOTUS's curation stages (harmonization, processing via lotus-processor,
  and automatic/manual validation that retained ~750,000 of 2.5M+ pairs) — unvalidated
  pairs should not be pushed to Wikidata
- You only need to query or export LOTUS data already hosted on Wikidata (use SPARQL/downloadLotus)
  rather than adding or updating entities
- The target is the LNPN mirror (lotus.naturalproducts.net / lotus-web) rather than
  Wikidata; the mirror does not use the Wikidata upsert path
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0219
tools:
- name: NPimporterBot
  role: Wikidata bot that executes the batch upload programs (npwdimporter), performing
    the upserts of structure-organism pair entities; its activity overview is used
    to validate uploads
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
- name: uploadLotus
  role: Module of lotus-wikidata-interact used to upload LOTUS data to Wikidata as
    upsert records
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
- name: downloadLotus
  role: Module used by uploadLotus to check what already exists on Wikidata before
    writing records
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
- name: SPARQL
  role: Query language used to check entity existence in Wikidata and to reconcile
    sampled exported pairs against live Wikidata entries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:6b4ff2ad4a32b756cdf97cdd21e6d01bad45fe0df29be6c180983fbc38a236b9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-wikidata-interact
  tools:
  - NPimporterBot
  - SPARQL
  - downloadLotus
  - uploadLotus
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# wikidata-property-mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Maps curated natural-product occurrence rows (chemical structure, organism, reference) onto the Wikidata properties used by LOTUS and prepares them as upsert records for batch upload via the lotus-wikidata-interact programs, checking for pre-existing entities first.

## When to use

Use when a curated, validated TSV of structure-organism-reference pairs (e.g. LOTUS table.tsv.gz from Zenodo 10.5281/zenodo.5794106) must be converted into Wikidata-formatted upsert records for batch import.

## When NOT to use

- Data has not yet passed LOTUS's curation stages (harmonization, processing via lotus-processor, and automatic/manual validation that retained ~750,000 of 2.5M+ pairs) — unvalidated pairs should not be pushed to Wikidata
- You only need to query or export LOTUS data already hosted on Wikidata (use SPARQL/downloadLotus) rather than adding or updating entities
- The target is the LNPN mirror (lotus.naturalproducts.net / lotus-web) rather than Wikidata; the mirror does not use the Wikidata upsert path

## Inputs

- Curated LOTUS table.tsv.gz of validated referenced structure-organism pairs (Zenodo deposit 10.5281/zenodo.5794106)
- SPARQL endpoint / Wikidata connection for entity existence checks
- lotus-wikidata-interact upload programs (uploadLotus, npwdimporter)

## Outputs

- wikidata_structure_organism_pairs.csv (Wikidata-format upsert records)
- Upserted or updated Wikidata entities for structures, organisms, references, and structure-organism pairs
- SPARQL query results reconciling sampled pairs against existing Wikidata entries

## How to apply

Load the curated LOTUS deposit (table.tsv.gz, DOI 10.5281/zenodo.5794106) with pandas and map each row's chemical structure, biological organism, and reference fields onto the corresponding Wikidata properties used by LOTUS. Before writing records, check whether the referenced structure, organism, and compound entities already exist in Wikidata — the downloadLotus module is explicitly used by uploadLotus for this existence check, via SPARQL or a direct connection. Group references, organisms, and compounds together (as the LOTUS Wikidata importer does) so that shared entities are reconciled rather than duplicated. Emit insert-or-update (upsert) records for every referenced structure-organism pair using the batch upload programs from the lotus-wikidata-interact repository (npwdimporter), and write the result to a Wikidata-format file such as wikidata_structure_organism_pairs.csv. The rationale is that Wikidata is LOTUS's primary hosting layer with community curation, so imports must reconcile against existing Q-items instead of blindly creating new ones.

## Related tools

- **NPimporterBot** (Wikidata bot that executes the batch upload programs (npwdimporter), performing the upserts of structure-organism pair entities; its activity overview is used to validate uploads) — https://github.com/lotusnprod/lotus-wikidata-interact
- **uploadLotus** (Module of lotus-wikidata-interact used to upload LOTUS data to Wikidata as upsert records) — https://github.com/lotusnprod/lotus-wikidata-interact
- **downloadLotus** (Module used by uploadLotus to check what already exists on Wikidata before writing records) — https://github.com/lotusnprod/lotus-wikidata-interact
- **SPARQL** (Query language used to check entity existence in Wikidata and to reconcile sampled exported pairs against live Wikidata entries)

## Examples

```
import pandas as pd; tbl = pd.read_csv('table.tsv.gz', sep='\t'); # map structure/organism/reference columns to LOTUS Wikidata properties, then run uploadLotus/npwdimporter upserts
```

## Evaluation signals

- wikidata_structure_organism_pairs.csv exists and is parseable after the export step
- Sampled pair records from the export can be reconciled against live Wikidata entries via SPARQL queries
- Entity-existence checks ran (via downloadLotus/SPARQL) before writing, so no duplicate Q-items are created for already-present structures, organisms, or references
- The NPimporterBot activity overview reflects the latest batch upload performed with these programs
- Only pairs that survived validation (structure, organism, and reference all described; ~750,000+ of 2.5M+ retained) appear in the exported records

## Limitations

- Wikidata lacks structural and spectral search modes, which is why LOTUS is dual-hosted; reconciliation of structures therefore relies on property mapping and SPARQL rather than structure search
- The lotus-wikidata-interact repository is archived; development continues at github.com/lotus-initiative
- The article reports no quantified accuracy for automated name-to-structure/structure-to-name translations, so property mappings derived from translated names may propagate those errors
- Some typical queries and editing workflows must fall back to manual data addition directly in Wikidata when the batch path does not cover them

## Evidence

- [readme] downloadLotus used by uploadLotus for existence checks: [evidence span withheld — anchor status: external_doc]
