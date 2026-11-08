---
name: twim-ccs-calibration-benchmarking
description: Use when evaluating whether a TWIM-MS CCS calibration approach correctly
  assigns features to biomolecular classes (small molecules, lipids, peptides) and
  produces accurate calibrated TWCCSN2 values, especially under simulated calibration
  error.
when_to_use_negative:
- Input is a single homogeneous class of analytes (e.g., pure peptide digest) with
  no need to discriminate between small molecule, lipid, and peptide calibrations
  — a single power law calibration suffices.
- Working with negative ionization mode data or chemical classes outside small molecules,
  lipids, and peptides; MOCCal's calibrations are not yet extended to these.
- Arrival times are drift-tube measured rather than traveling-wave; the class-assignment
  logic and power law fits are specific to TWIM-MS platforms.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: 'Tool under test: performs biomolecular class assignment and class-specific
    CCS calibration on perturbed or unperturbed TWIM-MS data'
  repo: https://github.com/HinesLab/MOCCal
- name: IMSCal
  role: Baseline calibration method for comparison against MOCCal's class-specific
    power law approach
- name: DEIMoS
  role: Used to extract arrival time data from raw .mzML calibration files and fit
    to a Gaussian distribution before MOCCal's power law parameter optimization
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:377cba20db3dca3fe61972060aeae228a68a452426c7ded2f2197e0c1eba5626
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - IMSCal
  - MOCCal
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# twim-ccs-calibration-benchmarking

## Summary

Benchmarking protocol for comparing TWIM-MS CCS calibration strategies (class-specific power laws vs. single power law vs. IMSCal) by quantifying class-assignment accuracy and calibrated CCS error under controlled calibration perturbations. It establishes error tolerances (±0.5–5.0%) and success thresholds (≥70% correct assignment) for tools like MOCCal.

## When to use

Use when evaluating whether a TWIM-MS CCS calibration approach correctly assigns features to biomolecular classes (small molecules, lipids, peptides) and produces accurate calibrated TWCCSN2 values, especially under simulated calibration error.

## When NOT to use

- Input is a single homogeneous class of analytes (e.g., pure peptide digest) with no need to discriminate between small molecule, lipid, and peptide calibrations — a single power law calibration suffices.
- Working with negative ionization mode data or chemical classes outside small molecules, lipids, and peptides; MOCCal's calibrations are not yet extended to these.
- Arrival times are drift-tube measured rather than traveling-wave; the class-assignment logic and power law fits are specific to TWIM-MS platforms.

## Inputs

- Perturbed or unperturbed CCS Compendium DTCCSN2 reference values (Supporting Information Document 2, Tables 1–3)
- Experimental feature table with m/z, charge state, retention time, and arrival time values for each feature
- Calibration data as .mzML files for class-specific power law parameter optimization
- Baseline calibration results from IMSCal and a single power law method for comparison

## Outputs

- Assignment accuracy percentages per class (small molecule, lipid, peptide) per calibration error level
- Comparison table (e.g., MOCCal_vs_IMSCal_vs_powerlaw_benchmark.tsv) with columns for method, class, calibration error level, and assignment accuracy
- Calibrated TWCCSN2 values with associated ΔCCS (Å²) to the assigned trendline
- Calibration effect scores (1–4 scale) quantifying how much the chosen calibration changes the result

## How to apply

Run the target calibration tool (e.g., MOCCal) on a complex multiomic extract such as a single-phase NIST SRM 1950 plasma extract, using the Unified CCS Compendium DTCCSN2 values as ground truth. To simulate calibration error, perturb the reference CCS values by ±0.5%, ±1.0%, ±2.0%, and ±5.0% and re-run the class-assignment workflow, which selects the calibration yielding the smallest ΔCCS (difference in Å² between calibrated TWCCSN2 and the DTCCSN2-vs-m/z regression line). Run IMSCal and a single power law calibration on the same perturbed data as baselines. Count correct assignments per class and error level: success is small molecules staying in the 81–83% assignment band at all error levels, and lipids/singly charged peptides maintaining ≥70% correct assignment at ±1% error.

## Related tools

- **MOCCal** (Tool under test: performs biomolecular class assignment and class-specific CCS calibration on perturbed or unperturbed TWIM-MS data) — https://github.com/HinesLab/MOCCal
- **IMSCal** (Baseline calibration method for comparison against MOCCal's class-specific power law approach)
- **DEIMoS** (Used to extract arrival time data from raw .mzML calibration files and fit to a Gaussian distribution before MOCCal's power law parameter optimization) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # within DEIMoS v1.3.2 virtual environment; or run MOCCal executable with a folder titled 'Output' present
```

## Evaluation signals

- Small-molecule assignment accuracy stays within the 81–83% band at every calibration error level (±0.5% to ±5.0%).
- Lipid and singly charged peptide assignment accuracy remains ≥70% at ±1% calibration error.
- For a known lipid feature (e.g., m/z 692.517), the class-matched power law yields calibrated CCS within ~0.18% of the DTCCSN2 value, versus ~1.8–2.6% error from mismatched class calibrations.
- Calibration effect scores range 1–4 (score 4 = ≥6% difference, score 3 = 3–6%, score 2 = 1–3%, score 1 = <1%); multiply charged assignments get default score 0.
- MOCCal's assignment accuracy exceeds IMSCal and a single power law calibration across the perturbation grid.

## Limitations

- Calibration errors confound feature identification if a single calibration equation is applied across multiomic features.
- MOCCal is currently limited to small molecule, lipid, and peptide calibrations on TWIM-MS platforms.
- User-driven expansion for negative ionization mode data sets and other endogenous or xenobiotic chemical classes is not yet implemented.
- The terms arrival time and drift time are used interchangeably in MOCCal, but TWIM platforms record arrival time (time reaching the detector), not true drift time — comparisons with DTCCSN2 reference values must account for this.

## Evidence

- [results] benchmark success criteria: "the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% with no clear influence of the positive or negative errors on the accuracy"
- [abstract] assignment mechanism: "uses the difference (in Å 2 ) between the calibrated TW CCS N2 value and DT CCS N2 vs m / z regression lines to determine the best calibration curve."
- [results] quantitative effect example: "the calibrated TWCCSN2 value from the lipid-fit power law is 0.18% from the DTCCSN2 value whereas the small molecule and peptide power laws yield calibrated TWCCSN2 values with 1.8 and 2.6% error"
- [results] scoring scheme: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match."
- [results] input pipeline: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
- [results] raw data pipeline: [evidence span withheld — anchor status: not_found]
- [results] benchmark test material: "We used single-phase extracts of a plasma reference material, NIST SRM 1950, to test the performance of MOCCal on a complex mixture"
- [discussion] scope boundary: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
- [readme] terminology caveat: "The terms arrival time and drift time are used interchangeably within the software for convenience. However, it should be noted that TWIM platforms record the time at which the ion reaches the"
