---
name: explained-feature-count-verification
description: Use when rerunning khipu annotation with modified isotope/adduct search
  patterns and you need to verify that the new run explains more features without
  inflating the empirical compound count (1,775 ESI+ / 908 ESI- expected on the yeast
  data).
when_to_use_negative:
- Input is raw LC-MS data rather than a preprocessed asari feature table — khipu requires
  a feature table as input
- The ESI- data has not been quality-filtered (SNR > 100), so the cleaner demo counts
  of 2,601/2,912 explained features would not be comparable
- Goal is metabolite identity assignment or benchmarking against libraries — khipu
  preannotation only infers neutral masses and empirical compounds, not definitive
  IDs
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: khipu
  role: Performs the annotation whose explained-feature and empirical-compound counts
    are being verified; run with default then user-supplied extended adduct patterns
  repo: https://github.com/shuzhao-li/khipu
- name: asari 1.9.2
  role: Produced the feature tables (yeast ESI+/ESI-) that the counting is performed
    on
  repo: https://github.com/shuzhao-li/asari
- name: Jupyter
  role: Demo notebooks where the baseline and extended-adduct annotation runs are
    executed before counting
- name: Python 3
  role: Language in which khipu is implemented and the count-verification script is
    written
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:4a42db68293181a929ae0dd1cdde5366fbf393798d5ef07bd63abfffe3c89b2e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Jupyter
  - Python 3
  - asari
  - asari 1.9.2
  - khipu
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# explained-feature-count-verification

## Summary

A validation skill for khipu annotation runs that counts explained (grouped, non-singleton) LC-MS features per ionization mode before and after an adduct-pattern expansion, checking that empirical compound counts stay stable while explained-feature counts reproduce reported values (e.g. 6,310→8,049 ESI+ and 2,601→2,912 ESI- on the Chen et al. yeast data).

## When to use

Use when rerunning khipu annotation with modified isotope/adduct search patterns and you need to verify that the new run explains more features without inflating the empirical compound count (1,775 ESI+ / 908 ESI- expected on the yeast data).

## When NOT to use

- Input is raw LC-MS data rather than a preprocessed asari feature table — khipu requires a feature table as input
- The ESI- data has not been quality-filtered (SNR > 100), so the cleaner demo counts of 2,601/2,912 explained features would not be comparable
- Goal is metabolite identity assignment or benchmarking against libraries — khipu preannotation only infers neutral masses and empirical compounds, not definitive IDs

## Inputs

- asari 1.9.2 feature tables for yeast ESI+ and SNR>100-filtered ESI- data (MassIVE MSV000087434, Chen et al.), tab delimited with feature ID, m/z, rtime, then intensity columns
- khipu annotation outputs (JSON and tab-delimited empCpd files) from baseline and extended-adduct runs

## Outputs

- Per-mode counts of explained (grouped, non-singleton) features before and after adduct expansion
- Per-mode empirical compound counts (expected 1,775 ESI+ and 908 ESI-)
- Summary table yeast_extended_adduct_annotation_counts.csv

## How to apply

Run the baseline khipu mass-difference search with built-in isotope patterns (e.g. 1.0034 Da for 13C/12C) and base adducts (e.g. 22.9893 Da for Na+) on the asari 1.9.2 feature tables, then rerun in the Jupyter demo notebooks with the additional Chen et al. adducts appended to user-supplied search patterns for both ESI+ and ESI- modes. After each run, count features that are grouped into non-singleton empirical compounds per ionization mode. Record counts before and after the adduct expansion and the empirical compound totals per mode. Correctness is judged by reproducing the reported explained-feature gains (6,310→8,049 ESI+; 2,601→2,912 ESI-) while empirical compound counts remain unchanged (1,775 and 908), then writing the summary to yeast_extended_adduct_annotation_counts.csv.

## Related tools

- **khipu** (Performs the annotation whose explained-feature and empirical-compound counts are being verified; run with default then user-supplied extended adduct patterns) — https://github.com/shuzhao-li/khipu
- **asari 1.9.2** (Produced the feature tables (yeast ESI+/ESI-) that the counting is performed on) — https://github.com/shuzhao-li/asari
- **Jupyter** (Demo notebooks where the baseline and extended-adduct annotation runs are executed before counting)
- **Python 3** (Language in which khipu is implemented and the count-verification script is written)

## Examples

```
python3 -m khipu.main -i yeast_pos.tsv -m pos -s 4 -o yeast_pos_baseline  # then rerun with extended adducts in the demo notebook and compare non-singleton feature counts
```

## Evaluation signals

- Extended-adduct run reproduces 8,049 explained ESI+ features and 2,912 explained ESI- features
- Baseline counts match 6,310 (ESI+) and 2,601 (ESI-) before expansion
- Empirical compound counts remain unchanged at 1,775 (ESI+) and 908 (ESI-) after the adduct expansion
- Output summary file yeast_extended_adduct_annotation_counts.csv contains the six recorded counts (before/after per mode plus compound counts)
- Feature-table integrity check: same input feature lists used for both runs (3,602 LC-MS features in the E. coli ESI+ reference case), so count differences stem only from pattern changes

## Limitations

- Benchmarking of khipu is deferred to future work, so explained-feature counts are internal consistency checks rather than accuracy metrics
- Fewer than 2,000 compounds were reliably measured in the examined experiments; explained-feature gains reflect adduct grouping, not new biology
- Counts depend on the SNR>100 filter for the ESI- demo data; unfiltered data will not reproduce the reported numbers

## Evidence
