---
name: cross-method-calibration-comparison
description: Use when you need to quantify how robust a CCS calibration/feature-class-assignment
  pipeline is to calibration error and whether it outperforms single-trendline or
  IMSCal baselines.
when_to_use_negative:
- You only need routine CCS calibration of already-identified features; use MOCCal's
  standard calibration path rather than this robustness benchmark.
- Your data are in negative ionization mode or belong to chemical classes outside
  small molecules, lipids, and peptides — MOCCal is not yet extended to these and
  the benchmark's class constraints (m/z < 550 / > 300) would not apply.
- You are comparing calibration methods on drift-tube rather than traveling-wave TWIM
  data, where the power-law calibration equations and trendline-ΔCCS assignment logic
  do not transfer.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: Performs the class assignment and multi-power-law TWCCSN2 calibration whose
    accuracy is benchmarked; runs calibration parameter determination, biomolecular
    class assignment, and structurally matched CCS calibrations without prior feature
    identification
  repo: https://github.com/HinesLab/MOCCal
- name: IMSCal
  role: Alternative calibration method run on the identical perturbed reference values
    to serve as the direct comparison baseline
- name: DEIMoS
  role: Extracts arrival-time data from raw calibration .mzML files and fits Gaussian
    distributions to supply the arrival times used for calibration parameter determination
  repo: http://github.com/pnnl/deimos
- name: CCS Compendium / LipidPioneer
  role: Reference databases of exact masses and DTCCSN2 values used to evaluate features
    and provide the perturbed reference CCS values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:94cc4ca54c60b157317c8b785b0d250ef98e51b2dac54f9b0a74e56b46492eb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - CCS Compendium / LipidPioneer
  - DEIMoS
  - IMSCal
  - MOCCal
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-method-calibration-comparison

## Summary

Benchmark a multiomic CCS calibration workflow against alternative calibration strategies (IMSCal and a single power law) by perturbing reference CCS values with simulated TWIM calibration errors (±0.5–5.0%) and measuring per-class assignment accuracy on NIST SRM 1950.

## When to use

Use when you need to quantify how robust a CCS calibration/feature-class-assignment pipeline is to calibration error and whether it outperforms single-trendline or IMSCal baselines.

## When NOT to use

- You only need routine CCS calibration of already-identified features; use MOCCal's standard calibration path rather than this robustness benchmark.
- Your data are in negative ionization mode or belong to chemical classes outside small molecules, lipids, and peptides — MOCCal is not yet extended to these and the benchmark's class constraints (m/z < 550 / > 300) would not apply.
- You are comparing calibration methods on drift-tube rather than traveling-wave TWIM data, where the power-law calibration equations and trendline-ΔCCS assignment logic do not transfer.

## Inputs

- Unified CCS Compendium DTCCSN2 reference values (Supporting Information Document 2, Tables 1–3)
- Simulated calibration error levels: ±0.5%, ±1.0%, ±2.0%, ±5.0%
- Feature lists with m/z, charge state, retention time, and arrival time for small molecules, lipids, and singly charged peptides
- NIST SRM 1950 single-phase extract features as the accuracy test substrate
- IMSCal and single power law calibration implementations as baselines

## Outputs

- Per-class, per-error-level assignment-accuracy percentages
- Comparison table (e.g. MOCCal_vs_IMSCal_vs_powerlaw_benchmark.tsv) with method, class, calibration error level, and accuracy columns
- Ranking of MOCCal vs IMSCal vs single power law at each error level

## How to apply

Take Unified CCS Compendium DTCCSN2 reference values and inject simulated calibration errors of ±0.5%, ±1.0%, ±2.0%, and ±5.0%, then run the class-assignment logic (m/z < 550 for small molecules, > 300 for lipids; select the calibrated TWCCSN2 value with the smallest ΔCCS to its trendline) for small molecules, lipids, and singly charged peptides. Run the identical perturbed benchmark under IMSCal and under a single power law calibration to generate baselines. Count correct class assignments per class and error level and express them as assignment-accuracy percentages. Judge success against the paper's reported criteria: small-molecule assignment stays in the 81–83% band at every error level, and lipid/peptide assignment remains ≥70% at ±1% error, with best performance at ±0.5%. Assemble the results into a comparison table with columns for method (MOCCal / IMSCal / single power law), biomolecular class, calibration error level, and assignment accuracy, e.g. written to MOCCal_vs_IMSCal_vs_powerlaw_benchmark.tsv.

## Related tools

- **MOCCal** (Performs the class assignment and multi-power-law TWCCSN2 calibration whose accuracy is benchmarked; runs calibration parameter determination, biomolecular class assignment, and structurally matched CCS calibrations without prior feature identification) — https://github.com/HinesLab/MOCCal
- **IMSCal** (Alternative calibration method run on the identical perturbed reference values to serve as the direct comparison baseline)
- **DEIMoS** (Extracts arrival-time data from raw calibration .mzML files and fits Gaussian distributions to supply the arrival times used for calibration parameter determination) — http://github.com/pnnl/deimos
- **CCS Compendium / LipidPioneer** (Reference databases of exact masses and DTCCSN2 values used to evaluate features and provide the perturbed reference CCS values)

## Examples

```
python MOCCal_RawDT.py --calibration calibration.mzML --experimental experiment.csv --output Output/  # then compare against IMSCal and single-power-law baselines at ±0.5–5.0% injected error
```

## Evaluation signals

- Small-molecule assignment accuracy stays within the 81–83% band for every simulated error level (±0.5% to ±5.0%) regardless of error sign
- Lipid and singly charged peptide assignment accuracy remains ≥70% at ±1% calibration error
- Best lipid/peptide assignment accuracy is achieved at ±0.5% error, with positive errors for lipids and negative errors for peptides trending toward the nearest trendline boundary
- MOCCal's assignment accuracy exceeds both IMSCal and the single power law baseline at every error level tested
- Output table contains a row for each (method × biomolecular class × error level) combination and per-class accuracy values sum to the expected counts of assigned vs misassigned features

## Limitations

- MOCCal is limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints and does not cover other endogenous or xenobiotic chemical classes
- MOCCal has not yet been extended to negative ionization mode data sets
- Single-trendline calibrations introduce calibration biases that can confound or prohibit feature identification, so a single power law baseline is expected to underperform at higher error levels
- Increasing positive errors for lipids and negative errors for peptides (≥±1%) trend toward trendline boundaries, degrading assignment accuracy beyond the ±0.5% optimum

## Evidence

- [results] the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% with no clear influence of the positive or negative errors on the accuracy: "the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% with no clear influence of the positive or negative errors on the accuracy"
- [results] MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [intro] The accuracy of the classifications and CCS calibrations has been tested against a single-phase extract of the human serum reference material, NIST SRM 1950: "The accuracy of the classifications and CCS calibrations has been tested against a single-phase extract of the human serum reference material, NIST SRM 1950"
- [results] The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match: "The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match"
- [results] less than m/z 550 for small molecules; greater than m/z 300 for lipids. If the feature falls outside the m/z constraints for the assigned class, the next closest match by ΔCCS will be selected: [evidence span withheld — anchor status: not_found]
