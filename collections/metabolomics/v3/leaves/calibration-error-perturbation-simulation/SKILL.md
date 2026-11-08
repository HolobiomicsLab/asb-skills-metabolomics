---
name: calibration-error-perturbation-simulation
description: Use when you need to quantify how sensitive a CCS class-assignment workflow
  (e.g. MOCCal vs IMSCal vs a single power law) is to systematic CCS calibration bias,
  by perturbing reference CCS values at fixed error levels.
when_to_use_negative:
- Your goal is to validate absolute calibrated TWCCSN2 accuracy against a known extract
  (e.g. NIST SRM 1950 identification), not to test robustness to calibration bias
  — use the direct accuracy benchmark instead.
- The features are already identified, so class assignment is unnecessary and only
  the class-specific power law equations should be applied directly.
- You are working in negative ionization mode or with chemical classes outside small
  molecules, lipids, and peptides — MOCCal's calibration equations do not yet cover
  these.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: 'Runs the class-assignment benchmark: perturbed reference features are calibrated
    with up to three class-specific power law equations and assigned via the smallest-ΔCCS
    rule.'
  repo: https://github.com/HinesLab/MOCCal
- name: IMSCal
  role: Baseline calibration method whose assignment accuracy under the same perturbed
    Compendium values is compared against MOCCal.
- name: DEIMoS
  role: Optional upstream step to extract Gaussian-apex arrival times from raw .mzML
    calibration files before the power law parameters used in the perturbation benchmark
    are fit.
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:f70e76b5daa034a617f7cf97cd39f53fd643d81aee5fa9f66a2bdf31e12fbeec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - IMSCal
  - MOCCal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# calibration-error-perturbation-simulation

## Summary

A robustness-testing method for multiomic TWIM CCS calibration that applies simulated calibration errors of ±0.5–5.0% to Unified CCS Compendium DTCCSN2 reference values and measures how correctly MOCCal reassigns features to their biomolecular class (small molecules, lipids, peptides).

## When to use

Use when you need to quantify how sensitive a CCS class-assignment workflow (e.g. MOCCal vs IMSCal vs a single power law) is to systematic CCS calibration bias, by perturbing reference CCS values at fixed error levels.

## When NOT to use

- Your goal is to validate absolute calibrated TWCCSN2 accuracy against a known extract (e.g. NIST SRM 1950 identification), not to test robustness to calibration bias — use the direct accuracy benchmark instead.
- The features are already identified, so class assignment is unnecessary and only the class-specific power law equations should be applied directly.
- You are working in negative ionization mode or with chemical classes outside small molecules, lipids, and peptides — MOCCal's calibration equations do not yet cover these.

## Inputs

- Unified CCS Compendium DTCCSN2 reference values (Supporting Information Document 2, Tables 1–3) for small molecules, lipids, and singly charged peptides
- Simulated calibration error levels: ±0.5%, ±1.0%, ±2.0%, ±5.0%
- m/z and charge state for each reference feature (used to enforce the small-molecule/lipid m/z constraints and to restrict peptide assignments to z = 1)
- Class-specific TWCCSN2 calibration parameters (from MOCCal's power law regressions on Waters Major Mix IMS/Tof Calibration Kit or Agilent ESI-L Tune Mix calibrants)

## Outputs

- Assignment accuracy table: percentage of correctly assigned features per biomolecular class (small molecules, lipids, peptides) at each error level (±0.5–5.0%) for each method (MOCCal, IMSCal, single power law), e.g. written as MOCCal_vs_IMSCal_vs_powerlaw_benchmark.tsv
- Per-feature assignment outcomes (correct class vs misassigned class) with the ΔCCS values that drove the selection

## How to apply

Start from Unified CCS Compendium DTCCSN2 values (Supporting Information Document 2, Tables 1–3) for small molecules, lipids, and singly charged peptides, and inject simulated TWIM CCS calibration errors of ±0.5%, ±1.0%, ±2.0%, and ±5.0% into the reference values as the paper does for its NIST SRM 1950 benchmark. For each error level, run MOCCal's class-assignment logic: enforce the m/z constraints (small molecules < m/z 550, lipids > m/z 300), compute ΔCCS between each calibrated TWCCSN2 value and the DTCCSN2-vs-m/z regression lines, and select the calibration whose value has the smallest ΔCCS to the corresponding trendline. In parallel, run the identical perturbed benchmark under IMSCal and under a single class-agnostic power law calibration so assignment accuracy can be compared across methods. Count correct assignments per biomolecular class and error level, then assemble a table with columns for method, class, error level, and assignment accuracy. Rationale: the perturbation isolates calibration bias from other error sources, because assignment correctness is driven purely by whether the smallest-ΔCCS rule still lands on the structurally matched class trendline when the reference values are systematically shifted.

## Related tools

- **MOCCal** (Runs the class-assignment benchmark: perturbed reference features are calibrated with up to three class-specific power law equations and assigned via the smallest-ΔCCS rule.) — https://github.com/HinesLab/MOCCal
- **IMSCal** (Baseline calibration method whose assignment accuracy under the same perturbed Compendium values is compared against MOCCal.)
- **DEIMoS** (Optional upstream step to extract Gaussian-apex arrival times from raw .mzML calibration files before the power law parameters used in the perturbation benchmark are fit.) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py --input perturbed_compendium.tsv --error_levels 0.5 1.0 2.0 5.0 --output MOCCal_vs_IMSCal_vs_powerlaw_benchmark.tsv
```

## Evaluation signals

- Small-molecule assignment accuracy stays within the reported 81–83% band at every error level from ±0.5% to ±5.0%, with no clear influence of error sign.
- Lipid and singly charged peptide assignment accuracy remains ≥70% at ±1% error, and is best at ±0.5% error.
- MOCCal's assignment accuracy at each error level exceeds that of both IMSCal and the single power law calibration.
- Every assignment respects the m/z constraints (< m/z 550 for small molecules, > m/z 300 for lipids); features falling outside the constraints fall through to the next closest ΔCCS match.
- The computed ΔCCS for the correctly assigned class is smaller than for the two competing calibrations for the majority of features at each error level.

## Limitations

- Coverage is restricted to small molecule, lipid, and peptide calibrations within TWIM-MS platforms; other endogenous or xenobiotic chemical classes are not yet included.
- Negative ionization mode data sets are not yet supported in MOCCal, so the perturbation benchmark is limited to positive-mode assignments.
- At ±5% error the correct assignment for lipids and peptides degrades — only small molecules remain stable in the 81–83% band, so the benchmark's claims for lipids/peptides are bounded at ±1% error.

## Evidence

- [results] the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% with no clear influence of the positive or negative errors on the accuracy: "the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% with no clear influence of the positive or negative errors on the accuracy"
- [results] MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [results] less than m/z 550 for small molecules; greater than m/z 300 for lipids. If the feature falls outside the m/z constraints for the assigned class, the next closest match by ΔCCS will be selected: [evidence span withheld — anchor status: not_found]
- [methods] Parameters are optimized for class-specific power law regressions using the calibrant's arrival time and reference drift tube CCS value: [evidence span withheld — anchor status: not_found]
- [discussion] User-driven expansion of MOCCal for the calibration of negative ionization mode data sets and the incorporation of other endogenous or xenobiotic chemical classes is welcomed: "User-driven expansion of MOCCal for the calibration of negative ionization mode data sets and the incorporation of other endogenous or xenobiotic chemical classes is welcomed"
