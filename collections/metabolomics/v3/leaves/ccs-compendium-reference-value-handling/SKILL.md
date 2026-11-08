---
name: ccs-compendium-reference-value-handling
description: Use when calibrating traveling-wave ion mobility arrival times or assigning
  unknown features to biomolecular classes and you need trusted DTCCSN2 reference
  values for small molecules, lipids, or peptides.
when_to_use_negative:
- Features are multiply charged — MOCCal assigns these a default calibration score
  of 0 and the Compendium trendline-matching logic is tailored to singly charged assignments
- The platform is not a Waters TWIM-MS instrument reporting arrival times (the Compendium
  DTCCSN2 values and power law framework here are TWCCSN2-specific)
- An analyte class outside small molecules, lipids, or peptides is targeted — MOCCal
  is scoped to these three calibrations and does not yet handle negative ionization
  mode data or other endogenous/xenobiotic classes
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MOCCal
  role: consumes Compendium DTCCSN2 values to optimize class-specific power law calibration
    parameters and build ΔCCS trendlines for class assignment
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: extracts calibrant arrival-time data (Gaussian apex) that are paired with
    Compendium reference CCS values during power law fitting
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:1a7b915cae7e9e8d753720138fe9757d29809119d1376bd9c453a0f57de4a708
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - CCS Compendium
  - DEIMoS
  - IMSCal
  - MOCCal
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ccs-compendium-reference-value-handling

## Summary

Prepares drift-tube CCS reference values (DTCCSN2) from the Unified CCS Compendium for use as calibration anchors in TWIM-MS power law regressions and as trendline targets for biomolecular class assignment.

## When to use

Use when calibrating traveling-wave ion mobility arrival times or assigning unknown features to biomolecular classes and you need trusted DTCCSN2 reference values for small molecules, lipids, or peptides.

## When NOT to use

- Features are multiply charged — MOCCal assigns these a default calibration score of 0 and the Compendium trendline-matching logic is tailored to singly charged assignments
- The platform is not a Waters TWIM-MS instrument reporting arrival times (the Compendium DTCCSN2 values and power law framework here are TWCCSN2-specific)
- An analyte class outside small molecules, lipids, or peptides is targeted — MOCCal is scoped to these three calibrations and does not yet handle negative ionization mode data or other endogenous/xenobiotic classes

## Inputs

- Unified CCS Compendium DTCCSN2 values for small molecules, lipids, and singly charged peptides (Supporting Information Document 2, Tables 1–3)
- Calibrant arrival times (Gaussian-apex values from DEIMoS extraction) to pair with reference CCS values

## Outputs

- Class-specific power law regression parameters relating arrival time to DTCCSN2
- ΔCCS vs m/z trendlines for small molecules, lipids, and peptides used for class assignment
- Perturbed reference sets for calibration-error simulation (±0.5–5.0%)

## How to apply

Pull the class-specific DTCCSN2 values from the Unified CCS Compendium as tabulated in Supporting Information Document 2 (Tables 1–3) rather than ad hoc literature CCS values, since MOCCal optimizes class-specific power law regressions using the calibrant's reference drift tube CCS value. First use the Compendium values as y-targets when fitting the power law (reduced mass–corrected) against Gaussian-fitted calibrant arrival times extracted via DEIMoS. Then build ΔCCS vs m/z trendlines from the same Compendium data — these trendlines define the smallest-ΔCCS decision rule by which MOCCal selects the correct class calibration for unknown features, subject to m/z constraints (<550 for small molecules, >300 for lipids). The Compendium values can also be artificially perturbed (±0.5% to ±5.0%) to stress-test assignment accuracy under simulated calibration errors. Verify the reference set spans all three biomolecular classes used by MOCCal's three calibration equations.

## Related tools

- **MOCCal** (consumes Compendium DTCCSN2 values to optimize class-specific power law calibration parameters and build ΔCCS trendlines for class assignment) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (extracts calibrant arrival-time data (Gaussian apex) that are paired with Compendium reference CCS values during power law fitting) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # in the DEIMoS venv; calibration data loaded as .mzML, Compendium DTCCSN2 values paired with Gaussian-apex arrival times
```

## Evaluation signals

- Small-molecule features from the Compendium should be assigned to small-molecule calibrations 81–83% of the time under ±0.5–5.0% perturbation with no strong dependence on error sign
- At ±0.5% error MOCCal should achieve correct assignment for ≥70% of lipids and singly charged peptides; at ±1% it should still reach ≥70%
- A known lipid feature calibrated with the lipid power law should sit ~0.18% from its DTCCSN2 value versus 1.8–2.6% when misassigned to small molecule or peptide equations
- The ΔCCS of the best-assigned class should fall within the m/z window bounds before selection (e.g. <550 for small molecules, >300 for lipids)

## Limitations

- MOCCal is currently limited to small molecule, lipid, and peptide calibrations within TWIM-MS constraints
- User-driven expansion for negative ionization mode data sets and other chemical classes is not yet implemented
- Reference values must come from the Unified CCS Compendium (DTCCSN2); using other CCS sources may shift the ΔCCS trendlines and degrade class assignment accuracy

## Evidence

- [results] Compendium values as calibration anchors: "regression lines developed from DTCCSN2 values in the Unified CCS Compendium (Supporting Information Document 2, Tables 1–3)"
- [results] power law optimization uses reference DTCCS: [evidence span withheld — anchor status: not_found]
- [results] class assignment via ΔCCS trendlines: "MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters"
- [results] assignment accuracy under perturbation: "the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83% with no clear influence of the positive or negative errors on the accuracy"
- [results] lipid/peptide assignment performance: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides"
- [results] m/z constraints for assignment: [evidence span withheld — anchor status: not_found]
- [discussion] MOCCal scope: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
- [discussion] negative mode not supported: "User-driven expansion of MOCCal for the calibration of negative ionization mode data sets and the incorporation of other endogenous or xenobiotic chemical classes is welcomed"
