---
name: chemical-identifier-reconciliation
description: Use when converting a curated tabular natural-products dataset (structure,
  organism, reference columns) into Wikidata entries and you must determine which
  entities already exist before writing.
when_to_use_negative:
- The goal is structural or spectral search over the dataset — the article notes these
  query modes are not available in Wikidata, which is why LOTUS is mirrored in parallel
  at lotus.naturalproducts.net
- The identifiers are not destined for Wikidata (e.g. purely local identifier normalization
  with no knowledge-base upsert target)
- Records have not been curated/validated yet — reconciliation should run on the validated
  ~30% subset, not raw pairs
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3315
tools:
- name: NPimporterBot
  role: Wikidata bot that executes the batch upsert programs against live Wikidata
    entities
- name: SPARQL
  role: Queries Wikidata to check whether chemical structure, organism, and reference
    entities already exist before upserting
- name: lotus-wikidata-interact (npwdimporter)
  role: Repository holding the batch upload programs (uploadLotus/downloadLotus) that
    perform the existence check and upsert; WDKt Kotlin library used by the uploader
  repo: https://github.com/lotusnprod/lotus-wikidata-interact
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:7c194239989f1b203aebf6332601e4d3a3263e4accab5fb963a92d8e8a4f2bed
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-wikidata-interact
  tools:
  - NPimporterBot
  - SPARQL
  - lotus-wikidata-interact (npwdimporter)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chemical-identifier-reconciliation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconciles curated natural-product structure-organism-reference records against existing Wikidata entities before generating upsert records, so each chemical structure, organism, and reference is inserted or updated exactly once via the LOTUS batch importer.

## When to use

Use when converting a curated tabular natural-products dataset (structure, organism, reference columns) into Wikidata entries and you must determine which entities already exist before writing.

## When NOT to use

- The goal is structural or spectral search over the dataset — the article notes these query modes are not available in Wikidata, which is why LOTUS is mirrored in parallel at lotus.naturalproducts.net
- The identifiers are not destined for Wikidata (e.g. purely local identifier normalization with no knowledge-base upsert target)
- Records have not been curated/validated yet — reconciliation should run on the validated ~30% subset, not raw pairs

## Inputs

- Curated LOTUS table.tsv.gz from Zenodo deposit 10.5281/zenodo.5794106 (validated referenced structure-organism pairs)
- Per-row chemical structure, biological organism, and reference fields (e.g. DOI/PMID identifiers)
- Current state of Wikidata entities, obtained via SPARQL queries or the downloadLotus direct connection

## Outputs

- wikidata_structure_organism_pairs.csv — Wikidata-format upsert records for every referenced structure-organism pair
- Grouped entity sets for references, organisms, and compounds, split into insert vs. update (upsert) operations

## How to apply

Load the curated LOTUS table.tsv.gz (Zenodo deposit 10.5281/zenodo.5794106) with pandas and map each row's chemical structure, biological organism, and reference fields onto the Wikidata properties used by LOTUS. Group references, organisms, and compounds together so shared entities are reconciled once rather than per pair. For each candidate entity, check whether it already exists in Wikidata — either via SPARQL queries or a direct connection (the downloadLotus module serves this purpose and is reused by uploadLotus). Only then emit upsert records (insert or update as needed) using the batch upload programs from the lotus-wikidata-interact repository (npwdimporter), driven by NPimporterBot. Rationale: reconciliation before upserting prevents duplicate chemical structure, taxon, and publication entities in Wikidata while preserving community curation already done there.

## Related tools

- **NPimporterBot** (Wikidata bot that executes the batch upsert programs against live Wikidata entities)
- **SPARQL** (Queries Wikidata to check whether chemical structure, organism, and reference entities already exist before upserting)
- **lotus-wikidata-interact (npwdimporter)** (Repository holding the batch upload programs (uploadLotus/downloadLotus) that perform the existence check and upsert; WDKt Kotlin library used by the uploader) — https://github.com/lotusnprod/lotus-wikidata-interact

## Examples

```
import pandas as pd; df = pd.read_csv('table.tsv.gz', sep='\t')  # map structure/organism/reference to Wikidata properties, emit upserts to wikidata_structure_organism_pairs.csv
```

## Evaluation signals

- wikidata_structure_organism_pairs.csv exists and is parseable
- Sampled pair records reconcile against live Wikidata entries via SPARQL queries, consistent with the NPimporterBot activity overview
- Shared references, organisms, and compounds are grouped so each is reconciled/upserted once, not duplicated per pair
- No entity is written without a prior existence check (SPARQL or direct connection), avoiding duplicate Q-items
- Record counts align with the curated input: 750,000+ validated pairs retained out of 2.5M+ initial pairs (~30%)

## Limitations

- The LOTUS processing pipeline is heavy, includes many dependencies and is convoluted
- Upstream chemical sanitization (standardizing, fragment removal, uncharging via MolVS) can degrade correct salts, charged or dimeric input molecules before reconciliation
- Structural and spectral searches are not available in Wikidata, limiting downstream verification to SPARQL-style queries

## Evidence

- [readme] uploader-existence-check: [evidence span withheld — anchor status: not_found]
- [results] sparql-exploration: "The Wikidata knowledge base and the option to use elaborate SPARQL queries allow the exploration of the dataset"
- [results] upsert-programs: "Programs for data addition to Wikidata can be found in the repository lotus-wikidata-interact."
- [results] upsert-definition: "insert or update the entities as needed (i.e. upserting)"
- [results] bot-activity: "an overview of the latest activity performed by our NPimporterBot, using those programs."
- [abstract] mirroring-rationale: "some typical queries of molecular electronic NP resources such as structural or spectral searches are not yet available in Wikidata. To address these shortcomings, LOTUS is hosted in parallel at"
