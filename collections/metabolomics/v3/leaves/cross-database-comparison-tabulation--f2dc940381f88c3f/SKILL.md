---
name: cross-database-comparison-tabulation
description: Use when a paper reports content volumes for multiple NP databases (structure
  counts, structure-organism pairs, records) and you need a side-by-side comparison
  table from those figures.
when_to_use_negative:
- Article reports per-entry quality metrics (e.g. 97% true positives) rather than
  aggregate content volumes — a different skill is needed
- Databases report different entity types without stated equivalence (e.g. PubChem's
  100 M chemical structures vs structure-organism pairs) — comparison would be misleading
  without normalization
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0219
tools:
- name: Python csv/pandas
  role: Write the extracted per-database metric rows into a structured CSV table
- name: LOTUS
  role: Target database whose reported content volume (750,000+ referenced structure-organism
    pairs) is benchmarked against baselines
  repo: https://github.com/lotusnprod
- name: KNApSAcK
  role: Baseline database providing 50,000+ structures and 100,000+ structure-organism
    pairs for comparison
- name: NAPRALERT
  role: Baseline database providing over 900,000 records (inflated by duplicate citations)
    for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:f115ee14b6df17c8ad8e880e56262e78aae9bb34dc2ea2c89284102ca14e432c
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - KNApSAcK
  - LOTUS
  - NAPRALERT
  - Python csv/pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-database-comparison-tabulation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Skill for benchmarking a new natural-products knowledge base (e.g. LOTUS) against established resources (KNApSAcK, NAPRALERT, COCONUT, DNP, NPAtlas) by extracting each database's reported content metrics into a structured CSV table.

## When to use

Use when a paper reports content volumes for multiple NP databases (structure counts, structure-organism pairs, records) and you need a side-by-side comparison table from those figures.

## When NOT to use

- Article reports per-entry quality metrics (e.g. 97% true positives) rather than aggregate content volumes — a different skill is needed
- Databases report different entity types without stated equivalence (e.g. PubChem's 100 M chemical structures vs structure-organism pairs) — comparison would be misleading without normalization

## Inputs

- Article text (abstract/results) containing reported content metrics per database
- Database names mentioned in text (LOTUS, KNApSAcK, NAPRALERT, COCONUT, NPAtlas, DNP, PubChem, GBIF)

## Outputs

- CSV table with one row per database, columns for database name, reported metric, and reported value
- Quantified comparison of structure-organism pair counts (e.g. LOTUS 750,000+ vs KNApSAcK 100,000+ vs NAPRALERT 900,000+)

## How to apply

Parse the article text for sentences where database content volumes are reported, e.g. the abstract sentence stating LOTUS 'completed the first steps toward the harmonization, curation, validation and open dissemination of 750,000+ referenced structure-organism pairs' alongside 'KNApSAcK (Afendi et al., 2012) currently contains 50,000+ structures and 100,000+ structure-organism pairs' and 'In total, NAPRALERT contains over 900,000 such records'. Extract the numeric values with their metric type (pairs, structures, records) and record them per database name. Note the comparability caveat the paper itself flags: NAPRALERT's 900,000+ is inflated 'due to equivalent structure-organism pairs reported in different citations', so metric types are not directly interchangeable. Assemble rows for each database with columns for database name, reported metric, and reported value, then write to CSV via Python's csv or pandas module.

## Related tools

- **Python csv/pandas** (Write the extracted per-database metric rows into a structured CSV table)
- **LOTUS** (Target database whose reported content volume (750,000+ referenced structure-organism pairs) is benchmarked against baselines) — https://github.com/lotusnprod
- **KNApSAcK** (Baseline database providing 50,000+ structures and 100,000+ structure-organism pairs for comparison)
- **NAPRALERT** (Baseline database providing over 900,000 records (inflated by duplicate citations) for comparison)

## Examples

```
import pandas as pd; df = pd.DataFrame([{'database':'LOTUS','metric':'referenced structure-organism pairs','value':'750,000+'},{'database':'KNApSAcK','metric':'structure-organism pairs','value':'100,000+'},{'database':'NAPRALERT','metric':'records','value':'900,000+'}]); df.to_csv('lotus_benchmark.csv', index=False)
```

## Evaluation signals

- CSV contains exactly one row per database mentioned (e.g. three rows for LOTUS, KNApSAcK, NAPRALERT)
- Extracted values match verbatim figures in the text (750,000+, 50,000+/100,000+, 900,000+)
- Metric type per row distinguishes pairs vs structures vs records
- No database is missing from the table if its content volume was stated in the parsed sections

## Limitations

- NAPRALERT's 900,000+ record count is not directly comparable to LOTUS's 750,000+ pairs because equivalent structure-organism pairs reported in different citations are counted separately
- Reported figures are approximate lower bounds ('+', 'over'), not exact counts
- The article does not provide a normalized, de-duplicated pair count for NAPRALERT or KNApSAcK, so a true like-for-like pair comparison is not possible from the stated figures alone

## Evidence

- [intro] KNApSAcK contains 50,000+ structures and 100,000+ structure-organism pairs: [evidence span withheld — anchor status: not_found]
- [full_text] Workflow step: extract metrics, tabulate, write CSV: [evidence span withheld — anchor status: not_found]
