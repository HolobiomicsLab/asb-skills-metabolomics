---
name: multiomic-class-assignment-accuracy-evaluation
description: Use when you need to quantify class-assignment accuracy of a CCS calibration
  tool on multiomic TWIM-MS data (NIST SRM 1950, CCS Compendium references) or compare
  that accuracy against IMSCal and a single power law calibration.
when_to_use_negative:
- When the goal is calibrated TWCCSN2 value accuracy rather than class-assignment
  accuracy of unknown features
- When the platform is not traveling wave ion mobility (TWIM) and the data are not
  analyzed with the MOCCal class-assignment logic (m/z constraints <550 for small
  molecules, >300 for lipids)
- When evaluating negative ionization mode data or chemical classes outside small
  molecules, lipids, and peptides, since MOCCal's calibration set is limited to those
  classes and positive ionization
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Class-assignment engine being evaluated; assigns each feature to the calibrated
    TWCCSN2 value with the smallest ΔCCS to the corresponding trendline subject to
    m/z constraints
  repo: github.com/HinesLab/MOCCal
- name: IMSCal
  role: Baseline calibration method against which MOCCal's assignment accuracy is
    benchmarked
- name: DEIMoS
  role: Used for raw calibration data extraction in the RawDT workflow feeding the
    benchmark (arrival-time extraction, Gaussian apex fitting)
  repo: pnnl/deimos
- name: Unified CCS Compendium
  role: Source of reference DTCCSN2 values perturbed with simulated calibration errors
    to build the assignment benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:46f8c41e5c4f8d1e72ef237e429f54eed57062828eb3a654b782bd61bd2fbe78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEIMoS
  - IMSCal
  - MOCCal
  - Unified CCS Compendium
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multiomic-class-assignment-accuracy-evaluation

## Summary

Evaluates how well MOCCal assigns unknown TWIM-MS features to their correct biomolecular class (small molecules, lipids, peptides) by benchmarking class-assignment accuracy under simulated calibration errors and against IMSCal and single power law baselines.

## When to use

Use when you need to quantify class-assignment accuracy of a CCS calibration tool on multiomic TWIM-MS data (NIST SRM 1950, CCS Compendium references) or compare that accuracy against IMSCal and a single power law calibration.

## When NOT to use

- When the goal is calibrated TWCCSN2 value accuracy rather than class-assignment accuracy of unknown features
- When the platform is not traveling wave ion mobility (TWIM) and the data are not analyzed with the MOCCal class-assignment logic (m/z constraints <550 for small molecules, >300 for lipids)
- When evaluating negative ionization mode data or chemical classes outside small molecules, lipids, and peptides, since MOCCal's calibration set is limited to those classes and positive ionization

## Inputs

- Unified CCS Compendium reference DTCCSN2 values (Supporting Information Document 2, Tables 1–3) for small molecules, lipids, and peptides
- Simulated calibration error levels: ±0.5%, ±1.0%, ±2.0%, ±5.0% applied to the reference CCS values
- Single-phase extract of NIST SRM 1950 as the complex-mixture benchmark material
- MOCCal, IMSCal, and single power law calibration implementations

## Outputs

- Assignment-accuracy percentages per biomolecular class (small molecules, lipids, singly charged peptides) and per calibration-error level for MOCCal, IMSCal, and the single power law
- Benchmark comparison table (e.g. MOCCal_vs_IMSCal_vs_powerlaw_benchmark.tsv) with columns for method, class, calibration error level, and assignment accuracy

## How to apply

Take Unified CCS Compendium DTCCSN2 reference values (Supporting Information Document 2, Tables 1–3) and perturb them with simulated calibration errors of ±0.5%, ±1.0%, ±2.0%, and ±5.0% to stress the assignment logic. For each error level and each biomolecular class, run MOCCal's class-assignment logic: apply the m/z constraints (less than m/z 550 for small molecules; greater than m/z 300 for lipids) and select the calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding DTCCSN2 vs m/z trendline, falling back to the next closest match when m/z constraints are violated. Run the identical benchmark under IMSCal and a single power law calibration to establish baseline comparisons. Count correct assignments per class and error level and compute assignment accuracy percentages; for the small-molecule class the reproduced values should stay in the 81–83% band at every error level, and lipid/peptide assignment should stay ≥70% at ±1% error (best lipid/peptide performance at ±0.5%, where increasing positive errors for lipids and negative errors for peptides trend toward the nearest trendline boundary).

## Related tools

- **MOCCal** (Class-assignment engine being evaluated; assigns each feature to the calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline subject to m/z constraints) — github.com/HinesLab/MOCCal
- **IMSCal** (Baseline calibration method against which MOCCal's assignment accuracy is benchmarked)
- **DEIMoS** (Used for raw calibration data extraction in the RawDT workflow feeding the benchmark (arrival-time extraction, Gaussian apex fitting)) — pnnl/deimos
- **Unified CCS Compendium** (Source of reference DTCCSN2 values perturbed with simulated calibration errors to build the assignment benchmark)

## Examples

```
python MOCCal_RawDT.py --input calibration.mzML --reference CCS_Compendium.csv --errors 0.5,1.0,2.0,5.0
```

## Evaluation signals

- Small-molecule assignment accuracy should remain within the 81–83% band across all tested calibration errors (±0.5% to ±5.0%) with no clear influence of error sign
- Lipid and singly charged peptide assignment accuracy should be ≥70% at ±1% calibration error
- Best lipid/peptide assignment should occur at ±0.5% error; increasing positive errors for lipids and negative errors for peptides trend toward the nearest trendline boundary
- MOCCal should outperform both IMSCal and the single power law calibration in correct-class assignment counts at the same error levels
- Verify that the assignment uses the paper's decision rule: smallest ΔCCS to the corresponding DTCCSN2 vs m/z trendline, with the m/z constraints (<550 for small molecules, >300 for lipids) enforced before selection

## Limitations

- MOCCal's assignment benchmark is limited to small molecules, lipids, and singly charged peptides; other chemical classes are not covered
- MOCCal is not yet extended to negative ionization mode data sets, so the benchmark does not cover negative-mode assignments
- The accuracy evaluation assumes reference DTCCSN2 values are the ground truth; errors in the Compendium propagate into the simulated calibration-error benchmarks
- The benchmark uses simulated calibration errors, not real instrumental drift, so real-world assignment accuracy may differ

## Evidence

- [results] Small molecules assigned 81-83% under ±0.5-5.0% calibration errors: "the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% with no clear influence of the positive or negative errors on the accuracy"
- [results] MOCCal achieves ≥70% correct assignment for lipids and peptides at ±1% error: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [abstract] Decision rule: smallest ΔCCS to the trendline: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] m/z constraints for class assignment: [evidence span withheld — anchor status: not_found]
- [discussion] MOCCal limited to small molecule, lipid, and peptide calibrations: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
