---
name: lipid-class-annotation-level-matching
description: Use when a per-lipid LipidLynxX Linker cross-reference table must be
  scored for identifier coverage against integration databases, and class-level gaps
  (e.g. which of the 131 lipids lack LION/KEGG/ChEBI IDs) need to be summarized.
when_to_use_negative:
- The lipids are plant or bacterial — LipidLynxX's Linker scope covers mammalian lipids
  from 25 classes in five categories, and plant/bacterial lipids are explicitly not
  covered
- The goal is to normalize annotation levels (e.g. species vs. sum-composition) for
  cross-dataset comparison — that is the Equalizer module's job, not Linker coverage
  scoring
- Input is a raw identification-software export not yet converted to LipidLynxX IDs
  — run the Converter first; the Linker expects lipid abbreviations in supported notation
  styles
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidLynxX
  role: Linker module translates lipid annotations into identifiers for data-integration
    tools (LION, KEGG, ChEBI, Rhea, UniProtKB) whose per-lipid presence/absence is
    scored here
  repo: https://github.com/SysMedOs/LipidLynxX
- name: pandas
  role: Builds the binary presence/absence coverage matrix and aggregates unmatched
    identifiers by lipid class/category
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:2f3cfd94d77af158ff9a21d4af9df8d664909b95299a050da6b07f69c14bb68a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SysMedOs/LipidLynxX
  repo_url: https://github.com/SysMedOs/LipidLynxX
  tools:
  - Lion
  - LipidLynxX
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipid-class-annotation-level-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many lipids in a LipidLynxX Linker translation run obtain identifiers for each data-integration target (LION, KEGG, ChEBI, Rhea, UniProtKB) and aggregates unmatched identifiers by lipid class/category to expose systematically unmapped classes.

## When to use

Use when a per-lipid LipidLynxX Linker cross-reference table must be scored for identifier coverage against integration databases, and class-level gaps (e.g. which of the 131 lipids lack LION/KEGG/ChEBI IDs) need to be summarized.

## When NOT to use

- The lipids are plant or bacterial — LipidLynxX's Linker scope covers mammalian lipids from 25 classes in five categories, and plant/bacterial lipids are explicitly not covered
- The goal is to normalize annotation levels (e.g. species vs. sum-composition) for cross-dataset comparison — that is the Equalizer module's job, not Linker coverage scoring
- Input is a raw identification-software export not yet converted to LipidLynxX IDs — run the Converter first; the Linker expects lipid abbreviations in supported notation styles

## Inputs

- Per-lipid LipidLynxX Linker translation results with cross-reference identifier columns (CSV/JSON table, one row per lipid abbreviation)
- Lipid class/category annotation per lipid (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)

## Outputs

- linker_coverage_table.csv: binary 131 x 5 coverage matrix (rows = lipids, columns = LION, KEGG, ChEBI, Rhea, UniProtKB)
- Per-database unmatched-class summary table aggregating NaN identifiers by lipid class and category

## How to apply

Parse the Linker translation output produced by LipidLynxX (one entry per lipid abbreviation) and extract the cross-reference identifier columns for LION, KEGG, ChEBI, Rhea, and UniProtKB. Build a 131-row binary presence/absence coverage matrix in pandas, treating NaN identifiers as unmatched. Annotate each lipid with its class and category — the Linker scope covers mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols) — then aggregate unmatched (NaN) counts per class per database to detect systematically unmatched classes. Write the matrix and the per-database unmatched-class summary to linker_coverage_table.csv. Reference counts from the test set (98/131 LION, 130/131 KEGG compound IDs, 49/131 ChEBI) serve as sanity anchors for validating a correctly run Linker translation.

## Related tools

- **LipidLynxX** (Linker module translates lipid annotations into identifiers for data-integration tools (LION, KEGG, ChEBI, Rhea, UniProtKB) whose per-lipid presence/absence is scored here) — https://github.com/SysMedOs/LipidLynxX
- **pandas** (Builds the binary presence/absence coverage matrix and aggregates unmatched identifiers by lipid class/category)

## Examples

```
import pandas as pd; df = pd.read_csv('linker_results.csv'); cov = df[['LION','KEGG','ChEBI','Rhea','UniProtKB']].notna().astype(int); cov.to_csv('linker_coverage_table.csv')
```

## Evaluation signals

- Coverage matrix contains exactly 131 lipid rows and five database coverage columns (LION, KEGG, ChEBI, Rhea, UniProtKB)
- Per-class unmatched summary reconciles exactly with row/column sums of the per-lipid binary matrix
- Column totals match the reported test-set anchors: 98 LION, 130 KEGG compound, and 49 ChEBI matches out of 131 lipids
- Every NaN cell in the coverage matrix maps to exactly one class-count entry in the unmatched-class summary (no double counting or drops)
- Lipid rows classified only within the five supported categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)

## Limitations

- LipidLynxX covers mammalian lipids from 25 classes in five categories; plant and bacterial lipids are not covered, so unmatched results for such lipids reflect tool scope, not data errors
- Known Linker runtime issues: if Linker runs more than 300s or more than 30s per ID, restart LipidLynxX and retry; allow firewall access for the Linker to reach databases
- Rhea and UniProtKB coverage may be absent or sparse relative to LION/KEGG/ChEBI, so per-database summaries can be dominated by unmatched entries for these targets

## Evidence
