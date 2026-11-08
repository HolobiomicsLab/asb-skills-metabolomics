---
name: traveling-wave-ccs-calibration
description: Use when analyzing TWIM-MS feature tables (m/z, charge, retention time,
  arrival time) from mixed small-molecule/lipid/peptide extracts on instruments like
  the Waters SYNAPT XS, where accurate, class-matched calibrated CCS values are needed
  without prior feature identification.
when_to_use_negative:
- Data are from drift tube (DTIMS) or other non-traveling-wave ion mobility instruments,
  where MOCCal's TWIM-specific power law equations do not apply
- The study requires negative ionization mode calibration or chemical classes outside
  small molecule/lipid/peptide — MOCCal is currently limited to those three classes
  and is not yet extended to negative mode data sets
- Features are already identified and structurally matched CCS values can be assigned
  directly; the class-assignment layer would add unnecessary uncertainty
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: 'Executes the TWCCSN2 calibration: loads .mzML calibration data, fits class-specific
    power law parameters, imports the feature spreadsheet, and assigns each feature
    to the best calibration by ΔCCS'
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts arrival time distributions from calibration .mzML files and fits
    a Gaussian whose apex is recorded as the calibrant's arrival time prior to parameter
    optimization (version 1.3.2 cited in the article)
  repo: http://github.com/pnnl/deimos
- name: Waters SYNAPT XS
  role: Traveling-wave ion mobility mass spectrometer on which the flow-injection
    calibration and NIST SRM 1950 test data were acquired
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:93307f66837e121e254a1bd1ca18f82b7f916ad7031ce2bc5209a17b7f40a01f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  - Waters SYNAPT XS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# traveling-wave-ccs-calibration

## Summary

Calibrates traveling-wave ion mobility (TWIM) arrival times into accurate TWCCSN2 values by fitting class-specific power law regressions (small molecule, lipid, peptide) and assigning unknown features to the structurally matching calibration via ΔCCS to DTCCSN2-vs-m/z trendlines.

## When to use

Use when analyzing TWIM-MS feature tables (m/z, charge, retention time, arrival time) from mixed small-molecule/lipid/peptide extracts on instruments like the Waters SYNAPT XS, where accurate, class-matched calibrated CCS values are needed without prior feature identification.

## When NOT to use

- Data are from drift tube (DTIMS) or other non-traveling-wave ion mobility instruments, where MOCCal's TWIM-specific power law equations do not apply
- The study requires negative ionization mode calibration or chemical classes outside small molecule/lipid/peptide — MOCCal is currently limited to those three classes and is not yet extended to negative mode data sets
- Features are already identified and structurally matched CCS values can be assigned directly; the class-assignment layer would add unnecessary uncertainty

## Inputs

- Calibration solution raw data as .mzML files (e.g. Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix, poly-dl-alanine, MassPREP enolase/ADH digests, SPLASH Lipidomix)
- Experimental feature spreadsheet containing feature identifiers, m/z, charge state, retention time, and arrival time values
- Reference drift tube CCS values for calibrants (e.g. Unified CCS Compendium tables)

## Outputs

- Optimized class-specific power law calibration parameters (small molecule, lipid, peptide)
- Per-feature calibrated TWCCSN2 values with assigned biomolecular class and ΔCCS to the class trendline
- Class assignment rates and calibration error summaries (e.g. percent deviation from DTCCSN2)

## How to apply

Fit calibration parameters for up to three class-specific power law equations using calibrant arrival times (Gaussian apex extracted via DEIMoS from .mzML) against reference drift tube CCS values (e.g. from the Unified CCS Compendium). Import the experimental feature spreadsheet (feature identifiers, m/z, charge, retention time, arrival time) and calibrate every singly charged feature with all three class equations, enforcing the m/z class constraints (<550 for small molecules, >300 for lipids). Select the calibrated TWCCSN2 value whose ΔCCS to the corresponding DTCCSN2-vs-m/z regression line is smallest; if the feature falls outside its class's m/z constraint, take the next closest match by ΔCCS. Multiply charged features default to the peptide calibration; apply decision thresholds like the calibration effect score (score 4 = ≥6% difference between assigned and next-closest class calibration) when auditing assignments. The rationale is that each biomolecular class has distinct fit parameters, so a structurally matched power law (e.g. lipid at m/z 692.517 within 0.18% of DTCCSN2 vs 1.8%/2.6% under small-molecule/peptide equations) yields more accurate CCS values than a single global calibration.

## Related tools

- **MOCCal** (Executes the TWCCSN2 calibration: loads .mzML calibration data, fits class-specific power law parameters, imports the feature spreadsheet, and assigns each feature to the best calibration by ΔCCS) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts arrival time distributions from calibration .mzML files and fits a Gaussian whose apex is recorded as the calibrant's arrival time prior to parameter optimization (version 1.3.2 cited in the article)) — http://github.com/pnnl/deimos
- **Waters SYNAPT XS** (Traveling-wave ion mobility mass spectrometer on which the flow-injection calibration and NIST SRM 1950 test data were acquired)

## Examples

```
python MOCCal_RawDT.py  # run in the DEIMoS virtual environment after installing DEIMoS 1.3.2, with calibration .mzML files and the feature spreadsheet in the working directory
```

## Evaluation signals

- ADH/enolase digest peptide TWCCSN2 within 2.6% of reference drift tube values; Poly-Ala6 within 0.6%, Poly-Ala7 within 1.2%, and multiply charged polyalanine within 0.62%
- Small molecules from the CCS Compendium assigned to the small molecule calibrations 81–83% of the time; ≥70% correct assignment for lipids and peptides with calibration errors of ±1%
- Feature at m/z 692.517: lipid-parametrized calibration should be 0.18% from DTCCSN2, vs 1.8%/2.6% under small-molecule/peptide equations — a sanity check that class matching is working
- Calibration effect scores in the 1–4 range (4 = ≥6% difference between assigned class and next closest match; 1 = <1%) are populated for singly charged features; multiply charged assignments default to score 0
- Performance under ramped TW wave velocity is comparable to static 600 m/s condition — class assignment rates and calibration errors should not diverge between acquisition modes

## Limitations

- MOCCal is currently built around small molecule, lipid, and peptide calibrations and has not been extended to negative ionization mode data sets or other endogenous/xenobiotic chemical classes
- Applying a single calibration equation to multiomic data introduces calibration biases that may confound feature identification
- Multiply charged features receive a default calibration effect score of 0, limiting auditability of their assignments
- Features outside their class's m/z constraint fall back to the next closest match by ΔCCS, which may degrade accuracy for boundary-region analytes

## Evidence

- [results] Class-specific power law selection rationale: "each biomolecular class has distinct fit parameters for the power law calibration curve"
- [results] Lipid example accuracy: "the calibrated TWCCSN2 value from the lipid-fit power law is 0.18% from the DTCCSN2 value whereas the small molecule and peptide power laws yield calibrated"
- [results] Overall performance envelope: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and"
- [results] Class-assignment m/z constraints: [evidence span withheld — anchor status: not_found]
- [results] MOCCal workflow: "MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters"
- [results] Calibrant arrival time extraction: [evidence span withheld — anchor status: not_found]
- [readme] README installation guidance: "If you plan to use raw calibration files, you will need to install pnnl’s DEIMoS, version 1.3.2 (http://github.com/pnnl/deimos)"
- [discussion] Limitation scope: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
