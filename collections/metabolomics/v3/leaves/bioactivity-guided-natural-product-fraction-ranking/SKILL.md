---
name: bioactivity-guided-natural-product-fraction-ranking
description: Decision support for prioritizing crude or roughly-fractionated natural
  product extract fractions when measured bioactivity values (e.g., cancer cell IC50-type
  cytotoxicity) are available and only the most potent fractions should proceed to
  isolation.
when_to_use_negative:
- The fractions have no measured bioactivity (e.g., cytotoxicity) values — ranking
  cannot be guided by potency and should instead rely on NMR/MS2 annotation alone.
- The input is already a purified, isolated compound set rather than roughly-fractionated
  extract fractions — prioritization-by-cytotoxicity is moot once isolation is complete.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: SMART 2.0
  role: NMR-based machine learning (Small Molecule Accurate Recognition Technology)
    mixture analysis tool applied in parallel with cytotoxicity ranking to guide prioritization
    of fractions toward isolation
- name: molecular networking
  role: MS2-based molecular networking used as a complementary filter alongside cytotoxicity
    to direct which fractions were isolated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:c7c2cc66d14070e972fc9427137759a2ec4774af696ac209d851e89c6f7d4c17
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bioactivity-guided-natural-product-fraction-ranking

## Summary

Ranks roughly-fractionated natural product extracts (here, a marine cyanobacterial Symploca extract) by measured cancer cell cytotoxicity so the most potent fractions are prioritized for downstream isolation, as done alongside SMART 2.0 NMR annotation and MS2-based molecular networking in this study.

## When to use

Decision support for prioritizing crude or roughly-fractionated natural product extract fractions when measured bioactivity values (e.g., cancer cell IC50-type cytotoxicity) are available and only the most potent fractions should proceed to isolation.

## When NOT to use

- The fractions have no measured bioactivity (e.g., cytotoxicity) values — ranking cannot be guided by potency and should instead rely on NMR/MS2 annotation alone.
- The input is already a purified, isolated compound set rather than roughly-fractionated extract fractions — prioritization-by-cytotoxicity is moot once isolation is complete.

## Inputs

- Table of measured cancer cell cytotoxicity values (e.g., IC50-type values) for each roughly-fractionated Symploca extract fraction
- Optional: SMART 2.0 NMR annotation results and MS2-based molecular networking results per fraction, used as complementary prioritization guides

## Outputs

- fraction_prioritization.csv with one row per fraction, its measured cytotoxicity value, and its priority rank (most potent fraction first)

## How to apply

Load the table of measured cancer cell cytotoxicity values recorded for each roughly-fractionated Symploca extract fraction and sort fractions so the most cytotoxic ones (lowest IC50-type values) receive the highest priority rank, mirroring the bioactivity-guided prioritization used in the study. In the source workflow, this cytotoxicity filter operated in parallel with NMR-based SMART 2.0 annotation and MS2-based molecular networking to direct which fractions underwent subsequent isolation. Write the ranked table to fraction_prioritization.csv with one row per fraction containing its measured cytotoxicity value and its rank. Validate that every input fraction appears exactly once and that row ordering is monotonically consistent with measured cytotoxicity (most potent fraction first). The downstream target in this study was isolation and rapid identification of a new chimeric swinholide-like macrolide, symplocolide A, a structural hybrid between swinholide A and luminaolide B.

## Related tools

- **SMART 2.0** (NMR-based machine learning (Small Molecule Accurate Recognition Technology) mixture analysis tool applied in parallel with cytotoxicity ranking to guide prioritization of fractions toward isolation)
- **molecular networking** (MS2-based molecular networking used as a complementary filter alongside cytotoxicity to direct which fractions were isolated)

## Evaluation signals

- fraction_prioritization.csv exists and contains exactly one row per input fraction (no fractions dropped or duplicated).
- Row ordering is monotonically consistent with measured cytotoxicity, with the most potent fraction (lowest IC50-type value) first.
- Each output row includes the fraction identifier, its measured cytotoxicity value, and its assigned rank.
- The set of fractions prioritized for isolation agrees with the study's guiding filters (cytotoxicity plus SMART 2.0 and MS2-based molecular networking signals).

## Limitations

- Cytotoxicity is only one of three prioritization signals in the study; cytotoxicity ranking alone may miss fractions prioritized via SMART 2.0 NMR annotation or MS2-based molecular networking.
- The skill assumes a pre-existing roughly-fractionated extract; it does not cover the fractionation itself (the 'environmental Symploca extract was roughly fractionated' upstream step).

## Evidence

- [abstract] This environmental Symploca extract was roughly fractionated: "This environmental Symploca extract was roughly fractionated"
- [abstract] prioritized and guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS2-based molecular networking: "prioritized and guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS 2 -based molecular networking"
- [abstract] first application of the novel NMR-based machine learning tool 'Small Molecule Accurate Recognition Technology' (SMART 2.0): "first application of the novel NMR-based machine learning tool ‘Small Molecule Accurate Recognition Technology’ (SMART 2.0)"
- [abstract] This led to the isolation and rapid identification of a new chimeric swinholide-like macrolide, symplocolide A: "This led to the isolation and rapid identification of a new chimeric swinholide-like macrolide, symplocolide A"
