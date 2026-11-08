---
name: cytotoxicity-based-fraction-prioritization
description: Decision support for selecting which roughly-fractionated extract fractions
  to carry into isolation and dereplication when a table of measured cancer cell cytotoxicity
  values per fraction is available.
when_to_use_negative:
- Fractions have not been measured for cancer cell cytotoxicity (no bioactivity readout
  to rank on) — use MS2-based molecular networking or SMART 2.0 NMR features alone
  instead.
- Input is already a prioritized or isolated-compound list (e.g. annotated symplocolide
  A / swinholide A / samholides A–I records) rather than per-fraction bioactivity
  measurements.
- The extract is not roughly fractionated into discrete fractions, so there is no
  fraction set to rank.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: SMART 2.0
  role: NMR-based mixture-analysis tool applied downstream to the cytotoxicity-prioritized
    fractions to guide discovery of new natural products
- name: molecular networking
  role: MS2-based molecular networking applied alongside the cytotoxicity filter to
    guide the prioritized fractions toward isolation
- name: tensorflow
  role: Supports SMART 2.0 model execution on prioritized fractions (keras models
    converted to HDF5 TF2 models under tensorflow 2.3.0)
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:32b720c9938b21e5d85a75bc59bf9b840abc7a35662c85183273b4bc8b36b053
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SMART 2.0
  - atomic sort
  - docker
  - docker-compose
  - molecular networking
  - tensorflow
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cytotoxicity-based-fraction-prioritization

## Summary

Bioactivity-guided ranking of roughly fractionated natural product extracts (e.g., a marine cyanobacterium Symploca extract) by measured cancer cell cytotoxicity, so the most potent fractions are selected for downstream isolation, SMART 2.0 NMR analysis, and MS2-based molecular networking.

## When to use

Decision support for selecting which roughly-fractionated extract fractions to carry into isolation and dereplication when a table of measured cancer cell cytotoxicity values per fraction is available.

## When NOT to use

- Fractions have not been measured for cancer cell cytotoxicity (no bioactivity readout to rank on) — use MS2-based molecular networking or SMART 2.0 NMR features alone instead.
- Input is already a prioritized or isolated-compound list (e.g. annotated symplocolide A / swinholide A / samholides A–I records) rather than per-fraction bioactivity measurements.
- The extract is not roughly fractionated into discrete fractions, so there is no fraction set to rank.

## Inputs

- Table of measured cancer cell cytotoxicity values for each roughly-fractionated Symploca extract fraction
- List of fraction identifiers from the fractionated environmental marine cyanobacterial (Symploca) extract

## Outputs

- fraction_prioritization.csv: one row per fraction with measured cytotoxicity value and potency-based priority rank

## How to apply

Load the table of measured cancer cell cytotoxicity values recorded for each roughly-fractionated Symploca extract fraction and sort fractions by cytotoxic potency (lowest IC50-type values first), mirroring the study's guiding filter alongside SMART 2.0 and MS2-based molecular networking. Emit a fraction_prioritization.csv with one row per fraction, its measured cytotoxicity value, and its potency-based rank. Validate that every input fraction appears exactly once and that row ordering is monotonically consistent with measured cytotoxicity (most potent fraction first). Fractions at the top of this ranking are the candidates passed to SMART 2.0 mixture analysis and molecular-networking-guided isolation, which in this study recovered symplocolide A, a structural hybrid between swinholide A and luminaolide B, plus swinholide A and samholides A–I.

## Related tools

- **SMART 2.0** (NMR-based mixture-analysis tool applied downstream to the cytotoxicity-prioritized fractions to guide discovery of new natural products)
- **molecular networking** (MS2-based molecular networking applied alongside the cytotoxicity filter to guide the prioritized fractions toward isolation)
- **tensorflow** (Supports SMART 2.0 model execution on prioritized fractions (keras models converted to HDF5 TF2 models under tensorflow 2.3.0))

## Examples

```
python prioritize_fractions.py --input cytotoxicity_values.csv --sort ascending_ic50 --output fraction_prioritization.csv
```

## Evaluation signals

- fraction_prioritization.csv exists and contains exactly one row per input fraction.
- Row ordering is monotonically consistent with measured cytotoxicity: the most potent fraction (lowest IC50-type value) is ranked first.
- No fraction identifiers are dropped or duplicated relative to the input table.
- Ranked top fractions, when fed to SMART 2.0 and molecular networking, reproduce the reported annotation of symplocolide A (swinholide A / luminaolide B structural hybrid), swinholide A, and samholides A–I.

## Limitations

- Ranking relies entirely on measured cancer cell cytotoxicity; fractions lacking a cytotoxicity measurement cannot be prioritized by this skill alone and must rely on the complementary SMART 2.0 and MS2-based molecular networking filters.
- Prioritization is only the triage step — it does not itself assign structures; structure confirmation of isolates (e.g. symplocolide A) requires 1D/2D NMR and LC-MS2 analysis.

## Evidence
