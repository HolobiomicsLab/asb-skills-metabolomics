---
name: priority-rank-table-generation
description: Use when you have a table of fractions or candidate samples with measured
  potency/cytotoxicity values and need to rank them so the most potent entries guide
  which material proceeds to isolation or deeper analysis.
when_to_use_negative:
- The input has no per-fraction quantitative bioactivity measurement, so ranking would
  be arbitrary rather than cytotoxicity-guided
- The goal is molecular annotation/classification of a single structure (e.g. NP Classifier
  on SMILES) rather than prioritizing multiple fractions for isolation
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: SMART 2.0
  role: NMR-based machine learning tool applied to fractionated mixtures, providing
    annotation evidence that supplemented the cytotoxicity ranking to prioritize fractions
    for isolation
- name: molecular networking
  role: MS2-based molecular networking used alongside the cytotoxicity filter to guide
    which fractionated Symploca extract fractions were pursued
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:1a6fc187cbd8b4a2cdb40cc0f6cdd8ac44d46cc44150f20d902d4a817511b3e3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# priority-rank-table-generation

## Summary

Generates a ranked prioritization table of sample fractions (e.g. roughly-fractionated marine cyanobacterial Symploca extract fractions) by sorting on a measured bioactivity readout such as cancer cell cytotoxicity (IC50-type values), mirroring the bioactivity-guided prioritization used to direct downstream isolation in the symplocolide A study.

## When to use

Use when you have a table of fractions or candidate samples with measured potency/cytotoxicity values and need to rank them so the most potent entries guide which material proceeds to isolation or deeper analysis.

## When NOT to use

- The input has no per-fraction quantitative bioactivity measurement, so ranking would be arbitrary rather than cytotoxicity-guided
- The goal is molecular annotation/classification of a single structure (e.g. NP Classifier on SMILES) rather than prioritizing multiple fractions for isolation

## Inputs

- Table (CSV/TSV) of roughly-fractionated extract fractions with a measured cancer cell cytotoxicity column (IC50-type values per fraction)
- Optional supporting prioritization evidence per fraction, e.g. SMART 2.0 NMR annotations and MS2-based molecular networking results

## Outputs

- fraction_prioritization.csv with one row per fraction, its measured cytotoxicity value, and its priority rank (most potent fraction first)

## How to apply

Load the table of measured cancer cell cytotoxicity values recorded for each roughly-fractionated Symploca extract fraction. Sort the fractions by cytotoxic potency so the most cytotoxic fractions (lowest IC50-type values) receive the highest priority rank. Write one row per fraction containing the fraction identifier, its measured cytotoxicity value, and its rank to fraction_prioritization.csv. In the source study, this cytotoxicity-based ranking was combined with NMR-based SMART 2.0 annotations and MS2-based molecular networking to prioritize which fractionated extract material was carried forward to isolation, which yielded symplocolide A plus swinholide A and samholides A–I annotations.

## Related tools

- **SMART 2.0** (NMR-based machine learning tool applied to fractionated mixtures, providing annotation evidence that supplemented the cytotoxicity ranking to prioritize fractions for isolation)
- **molecular networking** (MS2-based molecular networking used alongside the cytotoxicity filter to guide which fractionated Symploca extract fractions were pursued)

## Examples

```
python rank_fractions.py --input cytotoxicity.csv --column ic50 --ascending --output fraction_prioritization.csv
```

## Evaluation signals

- fraction_prioritization.csv exists and parses after the run
- Every input fraction appears exactly once in the output (no dropped or duplicated fraction identifiers)
- Row ordering is monotonically consistent with measured cytotoxicity, with the most potent fraction (lowest IC50-type value) in the first row
- Each output row contains exactly three fields: fraction identifier, measured cytotoxicity value, and rank
- Ranks are a dense, gap-free integer sequence 1..N matching the sorted order

## Limitations

- Ranking reflects only the single measured cytotoxicity readout; the study's prioritization was multi-criteria (cytotoxicity plus SMART 2.0 and MS2 molecular networking), so a pure cytotoxicity sort may diverge from an expert-guided selection
- IC50 values are assay-dependent; fractions from different assays or with missing/above-range cytotoxicity values cannot be rank-ordered consistently without a defined tie/missing-value policy

## Evidence

- [abstract] prioritized and guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS2-based molecular networking: [evidence span withheld — anchor status: not_found]
