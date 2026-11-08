---
name: qtof-collision-energy-spectra-matching
description: Use when comparing predicted vs experimental QTOF ESI-MS/MS spectra across
  multiple collision energies (10/20/40 eV) in [M+H]+ or [M-H]- mode, to quantify
  prediction accuracy per metric.
when_to_use_negative:
- Spectra were acquired on instruments other than QTOI QTOF — the article notes the
  inability to handle non-QTOF instruments
- Collision energies other than the 10/20/40 eV ladder used in the Metlin benchmark
- Peak matching at mass accuracy coarser than 0.01 Da / 10 ppm would invalidate the
  match definition
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CFM-ID 4.0
  role: Generates the predicted ESI-MS/MS spectra at 10/20/40 eV that are matched
    against experimental spectra
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Metlin
  role: Source of the experimental QTOF ESI-MS/MS reference spectra used as matching
    targets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:f86391a862ecb2935c84617a92c58c057d19e3205db15e3662ea6e21e6270459
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  - Metlin
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# qtof-collision-energy-spectra-matching

## Summary

Scores predicted ESI-MS/MS spectra against experimental QTOF spectra collected at 10, 20, and 40 eV collision energies using Dice coefficient, Precision, Recall, and re-weighted Stein's Dot Product, after iterative peak-intensity post-processing.

## When to use

Use when comparing predicted vs experimental QTOF ESI-MS/MS spectra across multiple collision energies (10/20/40 eV) in [M+H]+ or [M-H]- mode, to quantify prediction accuracy per metric.

## When NOT to use

- Spectra were acquired on instruments other than QTOI QTOF — the article notes the inability to handle non-QTOF instruments
- Collision energies other than the 10/20/40 eV ladder used in the Metlin benchmark
- Peak matching at mass accuracy coarser than 0.01 Da / 10 ppm would invalidate the match definition

## Inputs

- Predicted ESI-MS/MS spectra (peak lists of m/z and intensity) from CFM-ID at 10, 20, and 40 eV
- Experimental Metlin QTOF ESI-MS/MS spectra ([M+H]+ and [M-H]-) at 10, 20, and 40 eV collision energies
- Fold assignment (10-fold cross-validation splits) for held-out metabolites

## Outputs

- Per-energy-level Dice, Precision, Recall, and re-weighted Stein's Dot Product values per ionization mode
- Averaged metrics across energy levels and folds (e.g., table1_metrics.csv)
- Percent improvement comparisons between model versions with 2-sided t-test p-values

## How to apply

First post-process each predicted spectrum by iteratively keeping the most intense peaks until the summed intensity reaches 80% of the total or 30 peaks are selected. Then define a peak match as an m/z difference < 0.01 Da or 10 ppm (whichever threshold is hit first) between predicted and experimental peaks. Score at each collision energy (10, 20, 40 eV) and ionization mode separately, computing the Dice coefficient (peak presence), Precision, Recall, and re-weighted Stein's Dot Product. Average metrics across all energy levels and cross-validation folds, then compute percent improvement between model versions (e.g., CFM-ID 4.0 vs 2.0/3.0) and test significance with 2-sided t-tests.

## Related tools

- **CFM-ID 4.0** (Generates the predicted ESI-MS/MS spectra at 10/20/40 eV that are matched against experimental spectra) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Metlin** (Source of the experimental QTOF ESI-MS/MS reference spectra used as matching targets)

## Examples

```
docker run wishartlab/cfmid cfm-id --predict --molecule metabolite.sdf --energy 10,20,40  # then score peaks vs Metlin spectra at <0.01 Da / 10 ppm
```

## Evaluation signals

- Peak matches use m/z difference < 0.01 Da or 10 ppm (whichever first) — verify match tolerance is applied consistently
- Post-processed predicted spectra retain peaks until summed intensity reaches 80% of total or 30 peaks are selected
- Dice/Dot Product improvements of CFM-ID 4.0 over 2.0/3.0 must be positive on both ionization modes and directionally consistent with Table 1 (~31.8% Dice / 26.7% Dot Product on [M+H]+; ~8% Dice / 20.6% Dot Product on [M-H]-)
- t-test p-values should be highly significant (reported: 1E-43 for [M+H]+, 3E-29 for [M-H]- on Dice)
- Metrics computed per energy level (10/20/40 eV) and averaged across folds, matching the benchmark table structure

## Limitations

- Matching is restricted to QTOF ESI-MS/MS data; non-QTOF instrument spectra are not handled
- Deeper fragmentation graphs introduce extra computational complexity
- Match tolerance (0.01 Da / 10 ppm) makes the Dice coefficient sensitive to small m/z calibration differences between predicted and experimental peak lists

## Evidence

- [results] Dice/Precision/Recall match definition: [evidence span withheld — anchor status: not_found]
- [methods] Post-processing rule: [evidence span withheld — anchor status: not_found]
- [results] Metric suite: [evidence span withheld — anchor status: not_found]
