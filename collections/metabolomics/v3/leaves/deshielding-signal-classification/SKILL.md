---
name: deshielding-signal-classification
description: Use when 1H chemical shifts have been predicted for a compound library
  (e.g., the Natural Products Atlas set) and you must quantify shielded/deshielded
  signal coverage to judge dereplication feasibility.
when_to_use_negative:
- Input is experimentally acquired spectra rather than predicted shifts — this statistic
  is defined over in silico predicted 1H signals
- The goal is dereplicating compound classes with exclusively shielded 1H signals
  (sterols, low MW isoprenoids, lipids, volatile terpenes), which this coverage statistic
  explicitly precludes
- High-accuracy shift prediction is required — current lightweight predictors lack
  sufficient accuracy for direct dereplication
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMRShiftDB2
  role: Source of the stereo-aware HOSE code model used to predict 1H chemical shifts
    for every structure in the compound set
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:9bdec7b49bbdb0288b742319a94b0f6e6ee6abe74c481c1e1877c9fd10cd067b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NMRShiftDB2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# deshielding-signal-classification

## Summary

Classifies predicted 1H NMR signals for a large natural product structure set as shielded vs. deshielded and reports the fraction of molecules with at least 10% deshielded signals and with exclusively shielded signals, establishing whether lightweight in silico shift prediction can support NMR-based dereplication coverage.

## When to use

Use when 1H chemical shifts have been predicted for a compound library (e.g., the Natural Products Atlas set) and you must quantify shielded/deshielded signal coverage to judge dereplication feasibility.

## When NOT to use

- Input is experimentally acquired spectra rather than predicted shifts — this statistic is defined over in silico predicted 1H signals
- The goal is dereplicating compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes), which this coverage statistic explicitly precludes
- High-accuracy shift prediction is required — current lightweight predictors lack sufficient accuracy for direct dereplication

## Inputs

- 27,844 microbial natural product structures from the Natural Products Atlas database
- Stereo-aware HOSE code 1H shift prediction model from NMRShiftDB2

## Outputs

- Percentage of molecules possessing at least 10% deshielded 1H signals
- Percentage of molecules containing exclusively shielded 1H signals
- Summary statistics CSV file of the two percentages

## How to apply

Load the structure set (27,844 microbial natural products from the Natural Products Atlas database) and predict 1H chemical shifts for every structure using the stereo-aware HOSE code model available from NMRShiftDB2. For each molecule, classify each predicted 1H signal as shielded or deshielded and compute the molecule's fraction of deshielded signals. Tally molecules possessing at least 10% deshielded 1H signals and molecules containing exclusively shielded 1H signals across the full set. Write the two percentages to a summary statistics CSV; rationale is that shielded-only molecules (e.g., sterols, low MW isoprenoids, lipids, volatile terpenes) fall outside the deshielding-enriched coverage needed for HSQC/TOCSY-based dereplication.

## Related tools

- **NMRShiftDB2** (Source of the stereo-aware HOSE code model used to predict 1H chemical shifts for every structure in the compound set)

## Examples

```
python stats_deshielded.py --shifts predicted_1h_shifts.csv --min-deshielded-frac 0.10 --total 27844 --out deshielded_summary.csv
```

## Evaluation signals

- Computed percentage of molecules with at least 10% deshielded 1H signals equals 94.9%
- Computed percentage of molecules with exclusively shielded 1H signals equals 0.4%
- Molecule count tallied equals the full 27,844-compound Natural Products Atlas input set
- Each per-molecule deshielded fraction lies in [0, 1] and every predicted 1H signal is assigned to exactly one shielded/deshielded class
- Both percentages are persisted to the summary statistics CSV file

## Limitations

- Only 0.4% of predicted molecules are exclusively shielded, so compound classes such as sterols, low MW isoprenoids, lipids, and volatile terpenes fall outside deshielding-based coverage
- Current 'lightweight' NMR predictors, including the HOSE code approach, do not have sufficient accuracy to be useful for direct dereplication tasks
- TOCSY cross-peaks where both F1 and F2 are below 2.5 ppm (the shielded quadrant) are excluded in the downstream MADByTE platform, so shielded-region signals are not usable

## Evidence
