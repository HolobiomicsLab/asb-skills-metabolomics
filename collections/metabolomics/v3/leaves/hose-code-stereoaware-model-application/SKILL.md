---
name: hose-code-stereoaware-model-application
description: Use when you need predicted 1H chemical-shift distributions across a
  large compound library to quantify what fraction of molecules contain deshielded
  (>~2.5 ppm quadrant) protons, e.g., to validate NMR platform coverage.
when_to_use_negative:
- Compound classes whose members carry exclusively shielded 1H signals (sterols, low-MW
  isoprenoids, lipids, volatile terpenes) — the deshielded-signal statistic and the
  platform it justifies preclude these classes
- Tasks requiring accurate absolute shift prediction for individual spectra — the
  paper notes current lightweight NMR predictors lack the accuracy for direct dereplication
  matching
- Analysis of experimental spectra rather than structure-based prediction — this skill
  operates on compound structures, not acquired NMR data
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
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
  source_content_hash: sha256:a941249950c69bd2cf7846002161dbd365c781513032c9e641b4f260ce6856aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NMRShiftDB2
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hose-code-stereoaware-model-application

## Summary

Applies the stereo-aware HOSE code 1H NMR predictor from NMRShiftDB2 to a large structure set (e.g., 27,844 microbial natural products from the Natural Products Atlas) to classify shielded vs. deshielded proton signals and compute set-level statistics used to justify NMR dereplication design choices.

## When to use

Use when you need predicted 1H chemical-shift distributions across a large compound library to quantify what fraction of molecules contain deshielded (>~2.5 ppm quadrant) protons, e.g., to validate NMR platform coverage.

## When NOT to use

- Compound classes whose members carry exclusively shielded 1H signals (sterols, low-MW isoprenoids, lipids, volatile terpenes) — the deshielded-signal statistic and the platform it justifies preclude these classes
- Tasks requiring accurate absolute shift prediction for individual spectra — the paper notes current lightweight NMR predictors lack the accuracy for direct dereplication matching
- Analysis of experimental spectra rather than structure-based prediction — this skill operates on compound structures, not acquired NMR data

## Inputs

- Structure set of 27,844 microbial natural products from the Natural Products Atlas database
- Stereo-aware HOSE code prediction model from NMRShiftDB2 (provided via Supplemental Information)

## Outputs

- Per-molecule 1H chemical shift predictions and per-molecule fraction of deshielded signals
- Summary statistics CSV reporting the percentage of molecules with exclusively shielded 1H signals (0.4%) and the percentage with at least 10% deshielded signals (94.9%)

## How to apply

Load the compound structures (e.g., the 27,844 microbial natural products exported from the Natural Products Atlas database) and, for every structure, predict the 1H chemical shifts using the stereo-aware HOSE code model available from NMRShiftDB2 (the model and its usage are supplied in the paper's Supplemental Information). For each molecule, classify its predicted 1H signals as shielded or deshielded — using the shielded/deshielded boundary consistent with the MADByTE platform's shielded quadrant cutoff of 2.5 ppm in the 1H dimension — and compute the fraction of deshielded signals per molecule. Tally the molecules with exclusively shielded signals and those with at least 10% deshielded signals across the full set, then write the two percentages (expected 94.9% and 0.4%) to a summary statistics CSV. The rationale is that this statistic bounds which compound classes an HSQC/TOCSY-based dereplication platform can detect, since the shielded TOCSY quadrant (<2.5 ppm) is excluded from analysis.

## Related tools

- **NMRShiftDB2** (Source of the stereo-aware HOSE code model used to predict 1H chemical shifts for every structure in the compound set)

## Evaluation signals

- The computed percentage of molecules possessing at least 10% deshielded 1H signals must reproduce the reported value of 94.9%
- The computed percentage of molecules containing exclusively shielded 1H signals must reproduce the reported value of 0.4%
- Every structure in the 27,844-compound Natural Products Atlas set must yield a predicted shift list (no silently dropped entries) before tallying
- The shielded/deshielded classification boundary must match the 2.5 ppm shielded-quadrant cutoff used by the MADByTE TOCSY exclusion rule

## Limitations

- Sterols, low molecular weight isoprenoids, lipids, and volatile terpenes with exclusively shielded 1H signals are precluded from downstream analysis by this design
- Current lightweight NMR predictors (including HOSE-code approaches at this scale) do not have sufficient accuracy for direct dereplication of unknown spectra
- Peak-picking algorithms perform poorly in the crowded shielded region of real spectra, which is part of why the shielded quadrant is excluded

## Evidence
