---
name: marine-cyanobacterial-extract-fractionation-context
description: Use when a roughly-fractionated marine cyanobacterial (or other natural
  product) extract has per-fraction cancer cell cytotoxicity measurements (e.g., IC50-type
  values) and you must rank fractions to steer downstream isolation work.
when_to_use_negative:
- Input is a single crude (unfractionated) extract with one bulk cytotoxicity value
  — there are no per-fraction measurements to rank.
- The target bioactivity is not cancer-cell cytotoxicity (e.g., antimicrobial or enzyme-inhibition
  screens) — ranking by cancer cell cytotoxicity would misdirect isolation.
- No bioactivity data exist and prioritization must rest on chemistry alone (e.g.,
  MS2-based molecular networking or SMART 2.0 annotation only).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3173
tools:
- name: SMART 2.0
  role: NMR-based deep learning mixture annotation applied to the fractionated extract
    as the co-filter alongside cancer cell cytotoxicity when prioritizing fractions
    for isolation
- name: molecular networking
  role: MS2-based molecular networking used jointly with cytotoxicity to guide which
    fractionated Symploca fractions to pursue for isolation
- name: NP Classifier
  role: Downstream annotation of SMILES strings for candidates emerging from prioritized
    fractions via the /classify?smiles= API
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:39f3dc056db6873d6b902fce90da6663b1a5d82ccaf57d7782ae7a3d8ff5216a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP Classifier
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# marine-cyanobacterial-extract-fractionation-context

## Summary

Ranks roughly-fractionated fractions of an environmental marine cyanobacterial (Symploca) extract by measured cancer cell cytotoxicity so the most potent fractions advance to isolation, replicating the study's bioactivity-guided prioritization. In the source work this cytotoxicity filter was applied alongside NMR-based SMART 2.0 and MS2-based molecular networking, which together surfaced symplocolide A, a chimeric swinholide-like macrolide.

## When to use

Use when a roughly-fractionated marine cyanobacterial (or other natural product) extract has per-fraction cancer cell cytotoxicity measurements (e.g., IC50-type values) and you must rank fractions to steer downstream isolation work.

## When NOT to use

- Input is a single crude (unfractionated) extract with one bulk cytotoxicity value — there are no per-fraction measurements to rank.
- The target bioactivity is not cancer-cell cytotoxicity (e.g., antimicrobial or enzyme-inhibition screens) — ranking by cancer cell cytotoxicity would misdirect isolation.
- No bioactivity data exist and prioritization must rest on chemistry alone (e.g., MS2-based molecular networking or SMART 2.0 annotation only).

## Inputs

- Table of measured cancer cell cytotoxicity values (IC50-type) recorded for each roughly-fractionated Symploca extract fraction
- Optionally, SMART 2.0 NMR mixture annotations and MS2-based molecular networking outputs per fraction as corroborating tri-filter evidence

## Outputs

- fraction_prioritization.csv — one row per fraction with fraction identifier, measured cytotoxicity value, and priority rank, ordered most potent first
- Prioritized shortlist of fractions to advance to downstream isolation/dereplication

## How to apply

Load the table of measured cancer cell cytotoxicity values recorded for each roughly-fractionated Symploca extract fraction and sort ascending by potency so the most cytotoxic (lowest IC50-type value) fraction receives rank 1, keeping one row per fraction. Treat cytotoxicity as one axis of a tri-filter decision: in the study, fractionation was 'prioritized and guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS2-based molecular networking', so cross-check top-ranked fractions against SMART 2.0 mixture annotations and molecular networking hits before committing to isolation. Write the ranked table to fraction_prioritization.csv carrying the fraction identifier, its measured cytotoxicity value, and its integer priority rank. Advance top-ranked fractions into isolation/dereplication; in the paper this isolated symplocolide A (a structural hybrid of swinholide A and luminaolide B confirmed by 1D/2D NMR and LC-MS2 analysis) and annotated swinholide A, samholides A–I, and new derivatives. Validate that the output file exists, every input fraction appears exactly once, and row ordering is monotonically consistent with measured cytotoxicity (most potent fraction first).

## Related tools

- **SMART 2.0** (NMR-based deep learning mixture annotation applied to the fractionated extract as the co-filter alongside cancer cell cytotoxicity when prioritizing fractions for isolation)
- **molecular networking** (MS2-based molecular networking used jointly with cytotoxicity to guide which fractionated Symploca fractions to pursue for isolation)
- **NP Classifier** (Downstream annotation of SMILES strings for candidates emerging from prioritized fractions via the /classify?smiles= API) — https://github.com/mwang87/NP-Classifier

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('fractions.csv'); df['rank']=df['ic50'].rank(method='first').astype(int); df.sort_values('rank').to_csv('fraction_prioritization.csv', index=False)"
```

## Evaluation signals

- fraction_prioritization.csv exists and parses; every input fraction appears exactly once (no dropped or duplicated rows)
- Rank 1 is assigned to the fraction with the lowest measured cytotoxicity value, and row ordering is monotonically consistent with cytotoxicity across the whole table
- Each output row carries both the measured cytotoxicity value and an integer priority rank column
- Top-ranked fractions agree with the co-filters: SMART 2.0 mixture annotations and MS2-based molecular networking hits for those fractions (tri-filter consistency, as in the study)
- Isolation from top-ranked fractions recovers the expected compound series (in the paper: symplocolide A, swinholide A, samholides A–I)

## Limitations

- Cytotoxicity ranking alone is insufficient in this study's design: prioritization was jointly guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS2-based molecular networking.
- Requires per-fraction bioactivity measurements; a crude unfractionated extract yields a single value and cannot be prioritized this way.
- Prioritization only selects fractions — structural confirmation of candidates still requires separate experiments (symplocolide A's hybrid planar structure needed 1D/2D NMR and LC-MS2 analysis).

## Evidence

- [intro] Classify natural products programmatically from SMILES strings via an API: [evidence span withheld — anchor status: not_found]
