---
name: calibrated-ccs-error-computation
description: Use when MOCCal has produced calibrated TWCCSN2 values for experimental
  features (e.g. ADH/enolase digests, polyalanine peptides spiked into NIST SRM 1950)
  and you need to quantify their accuracy against reference drift tube CCS values.
when_to_use_negative:
- When raw, unprocessed calibration files must first be extracted and Gaussian-fitted
  before calibration parameters exist (run the RawDT workflow with DEIMoS instead)
- When features are not singly charged — the three-candidate calibration and ΔCCS
  selection procedure applies to singly charged features; multiply charged assignments
  skip the class trendline comparison (default score 0)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Computes up to three class-specific calibrated TWCCSN2 values per feature
    and selects the best match via ΔCCS to class trendlines; the calibrated values
    are then compared to reference DTCCSN2 to compute percent errors.
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used upstream in the RawDT workflow to extract arrival times from .mzML calibration
    files and fit them to a Gaussian distribution, feeding the calibration parameters
    needed for this skill.
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:28080121dec010c6f3027628376e06160cc3b23c8f7ccb3cbf6051c6b9062a41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# calibrated-ccs-error-computation

## Summary

Computes percent differences between MOCCal-calibrated TWCCSN2 values and reference drift tube (DTCCSN2) values to validate calibration accuracy for peptides from protein digests and polyalanine calibrants.

## When to use

Use when MOCCal has produced calibrated TWCCSN2 values for experimental features (e.g. ADH/enolase digests, polyalanine peptides spiked into NIST SRM 1950) and you need to quantify their accuracy against reference drift tube CCS values.

## When NOT to use

- When raw, unprocessed calibration files must first be extracted and Gaussian-fitted before calibration parameters exist (run the RawDT workflow with DEIMoS instead)
- When features are not singly charged — the three-candidate calibration and ΔCCS selection procedure applies to singly charged features; multiply charged assignments skip the class trendline comparison (default score 0)

## Inputs

- Experimental feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values (e.g. FI-IM-MS data of NIST SRM 1950 spiked with polyalanine and protein digests)
- MOCCal class-specific power law calibration parameters (small molecule, lipid, peptide fits) derived from calibrant arrival times and reference drift tube CCS values
- Reference DTCCSN2 values from the Unified CCS Compendium for ADH/enolase digest peptides, Poly-Ala6, Poly-Ala7, and multiply charged polyalanine peptides

## Outputs

- Calibrated TWCCSN2 values for each feature (up to three candidates, then the selected best match)
- Percent difference between calibrated TWCCSN2 and reference DTCCSN2 for each target feature
- validation_ccs_values.csv containing computed calibrated values and percent differences

## How to apply

Load the experimental feature spreadsheet (feature identifiers, m/z, charge state, retention time, arrival time) and run MOCCal's class-specific power law calibrations (small molecule, lipid, peptide fits) to compute up to three candidate TWCCSN2 values per feature. Select the value with the smallest ΔCCS to the corresponding biomolecular class trendline, respecting m/z constraints (small molecules < 550, lipids > 300). Extract the calibrated CCS values for ADH digest, enolase digest, Poly-Ala6, Poly-Ala7, and doubly/triply charged polyalanine features, then compare each against its reference drift tube CCS value to compute percent difference. Verify against the reported error bounds: ADH/enolase digest peptides within 2.6%, Poly-Ala6 within 0.6%, Poly-Ala7 within 1.2%, and multiply charged polyalanine within 0.62%. Multiply charged assignments receive a default effect score of 0 since no class trendline comparison applies.

## Related tools

- **MOCCal** (Computes up to three class-specific calibrated TWCCSN2 values per feature and selects the best match via ΔCCS to class trendlines; the calibrated values are then compared to reference DTCCSN2 to compute percent errors.) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used upstream in the RawDT workflow to extract arrival times from .mzML calibration files and fit them to a Gaussian distribution, feeding the calibration parameters needed for this skill.) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # after installing pnnl's DEIMoS v1.3.2, per http://github.com/pnnl/deimos
```

## Evaluation signals

- Percent differences for doubly/triply charged polyalanine peptides fall within 0.62% of drift tube DTCCSN2 values
- Poly-Ala6 calibrated TWCCSN2 within 0.6% and Poly-Ala7 within 1.2% of their reference drift tube values
- ADH and enolase digest peptide percent differences do not exceed 2.6% of reference drift tube CCS
- Each feature's selected calibrated value has the smallest ΔCCS to its class trendline, and small-molecule features respect the m/z < 550 constraint while lipid features respect m/z > 300
- Output file validation_ccs_values.csv contains one row per feature with its calibrated TWCCSN2 and computed percent difference

## Limitations

- MOCCal is currently limited to small molecule, lipid, and peptide calibrations on TWIM-MS platforms; other biomolecular classes are not supported
- MOCCal does not yet handle negative ionization mode data sets or other endogenous/xenobiotic chemical classes without user-driven expansion
- Applying a single (non-class-matched) calibration equation introduces calibration biases that may confound feature identification

## Evidence

- [abstract] MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [results] The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match: "The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match"
- [results] the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83%: "the small molecules from the CCS Compendium are assigned to the small molecule calibrations 81–83%"
- [results] MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5%: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5%"
- [discussion] MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
