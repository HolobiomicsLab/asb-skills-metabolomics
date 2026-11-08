---
name: structured-csv-output-generation
description: Use when you have batch-processed MSThunder outputs from multiple water
  sample groups (influent, effluent, surface, source, tap water) and need a consolidated,
  deduplicated pollutant inventory with sample assignments for reporting or validation.
when_to_use_negative:
- Input is a single sample group or a pre-existing aggregated identification table
  with no multi-group deduplication needed
- The goal is spectral-level candidate ranking (candidate scores in MSThunder's Ranking
  section) rather than compiling a sample-level pollutant inventory
- Data come from matrices other than water samples or from vendor raw files not yet
  converted via MSConvert and processed through the MSThunder pipeline
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3520
tools:
- name: MSThunder
  role: Produces the per-sample pollutant identification records (precursor, retention
    time, ISMILES, candidate formula) that are aggregated into the CSV
  repo: https://github.com/LQZ0123/MSThunder
- name: pandas 1.5.0
  role: Used within the MSThunder Python environment for tabular data handling and
    writing the compiled CSV output
  repo: https://github.com/LQZ0123/MSThunder
- name: PubChemPy 1.0.4
  role: Converts SMILES to ISMILES sequences used as the structural annotation column
    in the compiled table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.enceco.2025.07.022
    title: 'MSThunder: A deep learning-based nontargeted analytical framework for
      the accurate and rapid identification of unknown organic pollutants in water'
  source_content_hash: sha256:110af18a6526be9025fa102a55f008433c2910bb97a489fd17cfea5180a7c18f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LQZ0123/MSThunder
  repo_url: https://github.com/LQZ0123/MSThunder
  tools:
  - MSThunder
  - PubChemPy
  - PubChemPy 1.0.4
  - Python 3.10.5
  - pandas 1.5.0
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# structured-csv-output-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compile and deduplicate MSThunder's pollutant identification results across multiple water sample groups into a single structured CSV table (e.g. yellow_river_pollutants.csv), annotating each entry with precursor ion, retention time, ISMILES/candidate structure, and sample-group assignments, so that total identification counts and first-reported hazardous pollutants can be verified against published claims.

## When to use

Use when you have batch-processed MSThunder outputs from multiple water sample groups (influent, effluent, surface, source, tap water) and need a consolidated, deduplicated pollutant inventory with sample assignments for reporting or validation.

## When NOT to use

- Input is a single sample group or a pre-existing aggregated identification table with no multi-group deduplication needed
- The goal is spectral-level candidate ranking (candidate scores in MSThunder's Ranking section) rather than compiling a sample-level pollutant inventory
- Data come from matrices other than water samples or from vendor raw files not yet converted via MSConvert and processed through the MSThunder pipeline

## Inputs

- MSThunder identification outputs for eight Yellow River Basin water sample groups (influent, effluent, 2x surface water, 2x source water, 2x tap water)
- Per-compound records containing precursor ion, retention time, ISMILES, and candidate precursor formula from the MSThunder Candidate/MS2 List sections

## Outputs

- yellow_river_pollutants.csv — structured table of 181 deduplicated pollutants with sample-group assignments, precursor ion, retention time, and ISMILES annotations
- Sub-list flagging the three first-reported pollutants (triallyl cyanurate, harmaline, 5,7-dimethyl-3-methylsulfonyl-[1,2]thiazolo[3,4-d]pyrimidine-4,6-dione) with their occurrence groups

## How to apply

Load the identified-compound records from each of the eight Yellow River Basin sample groups (one influent, one effluent, two surface water, two source water, two tap water samples) produced by the MSThunder workflow. Aggregate the records and deduplicate compounds that appear in multiple groups, keeping per-group occurrence mappings so the compiled table reflects all 181 unique identified pollutants reported in the abstract. Annotate each entry with its precursor ion, retention time, and candidate structure/ISMILES information as produced by the transformer-based prediction pipeline (e.g. triallyl cyanurate, harmaline). Explicitly verify the presence of the three first-reported hazardous pollutants and record which sample group each occurred in. Write the final table to a structured CSV (e.g. yellow_river_pollutants.csv) using pandas (version 1.5.0 in the MSThunder environment), then validate the row count and the identity of the first-reported compounds against the article's claims.

## Related tools

- **MSThunder** (Produces the per-sample pollutant identification records (precursor, retention time, ISMILES, candidate formula) that are aggregated into the CSV) — https://github.com/LQZ0123/MSThunder
- **pandas 1.5.0** (Used within the MSThunder Python environment for tabular data handling and writing the compiled CSV output) — https://github.com/LQZ0123/MSThunder
- **PubChemPy 1.0.4** (Converts SMILES to ISMILES sequences used as the structural annotation column in the compiled table)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('yellow_river_pollutants.csv'); assert len(df)==181"
```

## Evaluation signals

- Compiled CSV contains exactly 181 unique identified pollutants after deduplication across the eight sample groups
- Table includes the three first-reported pollutants: triallyl cyanurate, harmaline, and 5,7-dimethyl-3-methylsulfonyl-[1,2]thiazolo[3,4-d]pyrimidine-4,6-dione
- Every row has non-empty precursor ion, retention time, and ISMILES annotations consistent with the MSThunder Candidate section fields
- Sample-group assignment column reflects the eight-group design (1 influent, 1 effluent, 2 surface, 2 source, 2 tap water) with no unassigned compounds
- CSV parses cleanly with pandas and row count matches the deduplication log

## Limitations

- Counts depend on the upstream MSThunder model's candidate inference (77.1% structure accuracy on spiked samples vs SIRIUS 74.0% and TraceFinder 46.9%); misidentifications propagate into the CSV
- The three first-reported pollutants are only 'first found' relative to this study's water samples — absence of prior literature reporting should be re-verified
- Sample-group occurrence is limited to the eight Yellow River Basin groups; the table does not generalize to other basins or matrices
- MSThunder currently requires converted raw data processed in a Linux system before CSV compilation; offline raw-data processing is not yet supported

## Evidence
