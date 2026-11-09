---
name: calibration-error-metrics-compilation
description: Use when evaluating or reporting the accuracy of MOCCal class-assigned
  CCS calibrations on a multiomic TWIM-MS feature table, such as a spiked NIST SRM
  1950 plasma extract analyzed by flow-injection on a Waters SYNAPT XS.
when_to_use_negative:
- When the goal is raw feature detection or arrival-time extraction rather than post-hoc
  accuracy summarization — use DEIMoS feature detection/Gaussian apex fitting upstream
  of MOCCal instead.
- When calibrating only one biomolecular class (e.g. peptides only) without class
  assignment — a single power-law calibration does not require compiling per-class
  error metrics.
- When the input is raw .mzML calibration files rather than a processed feature table
  — the RawDT workflow (DEIMoS 1.3.2) should be used first to produce the arrival-time
  spreadsheet.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Produces the class-assigned CCS calibrations whose per-class error percentages
    and RMSE values are compiled into the summary CSV.
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts calibrant arrival times from .mzML raw data (UserDT workflow alternative)
    prior to MOCCal calibration; version 1.3.2 is required for the RawDT variant.
  repo: http://github.com/pnnl/deimos
- name: NIST SRM 1950
  role: Multiomic reference material (human plasma extract spiked with ADH/enolase
    digests) used as the test mixture whose per-class errors and RMSE are compiled.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:84b0c1fdfec03b6fc18e8e79306a3da585c2195947d566a1bd98df4b4b36b959
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - Agilent ESI-L Tune Mix
  - DEIMoS
  - MOCCal
  - NIST SRM 1950
  - Waters Major Mix IMS/Tof Calibration Kit
  - poly-dl-alanine
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# calibration-error-metrics-compilation

## Summary

Compiles the per-class calibration error percentages and root-mean-square errors (RMSE) reported by MOCCal's class-specific TWIM CCS calibrations into a single summary CSV for lipids, small molecules, and peptides at charge states z = 1, 2, and 3.

## When to use

Use when evaluating or reporting the accuracy of MOCCal class-assigned CCS calibrations on a multiomic TWIM-MS feature table, such as a spiked NIST SRM 1950 plasma extract analyzed by flow-injection on a Waters SYNAPT XS.

## When NOT to use

- When the goal is raw feature detection or arrival-time extraction rather than post-hoc accuracy summarization — use DEIMoS feature detection/Gaussian apex fitting upstream of MOCCal instead.
- When calibrating only one biomolecular class (e.g. peptides only) without class assignment — a single power-law calibration does not require compiling per-class error metrics.
- When the input is raw .mzML calibration files rather than a processed feature table — the RawDT workflow (DEIMoS 1.3.2) should be used first to produce the arrival-time spreadsheet.

## Inputs

- MOCCal 'Output' folder contents from the UserDT executable run (per-feature assignments and calibration errors)
- NIST SRM 1950 single-phase extract feature table (feature identifiers, m/z, charge state, retention time, arrival time)
- Calibrant spreadsheets: poly-dl-alanine peptides, Waters Major Mix IMS/Tof Calibration Kit, or Agilent ESI-L Tune Mix

## Outputs

- Single summary CSV with per-class calibration error percentages (0.02, 0.75, 0.01, 0.06, 0.13%) and RMSE values (0.28, 0.56, 0.09, 0.09, 0.24) for lipids, small molecules, and peptides z=1/2/3

## How to apply

1. Run MOCCal (UserDT executable) with the packaged calibrant data (poly-dl-alanine peptides, Waters Major Mix IMS/Tof Calibration Kit, or Agilent ESI-L Tune Mix) and the experimental feature table from the NIST SRM 1950 extract spiked with ADH/enolase digests; MOCCal writes its per-feature assignments into the 'Output' folder. 2. Parse the 'Output' folder contents to extract, for each biomolecular class (small molecules, lipids, peptides z=1/2/3), the calibration error percentage relative to the DTCCSN2 reference values and the RMSE of the calibrated TWCCSN2 values. 3. Assemble these numbers into a single summary CSV whose columns mirror the published table: percent errors 0.02% (lipids), 0.75% (small molecules), 0.01%/0.06%/0.13% (peptides z=1/2/3) and RMSE 0.28/0.56/0.09/0.09/0.24 in the same class order. 4. Validation is by direct comparison to the published table: percent errors must reproduce 0.02, 0.75, 0.01, 0.06, 0.13% and RMSE must reproduce 0.28, 0.56, 0.09, 0.09, 0.24 within the tolerance implied by that table; deviations beyond that tolerance indicate a mis-assigned class or an incorrect power-law parameter set.

## Related tools

- **MOCCal** (Produces the class-assigned CCS calibrations whose per-class error percentages and RMSE values are compiled into the summary CSV.) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts calibrant arrival times from .mzML raw data (UserDT workflow alternative) prior to MOCCal calibration; version 1.3.2 is required for the RawDT variant.) — http://github.com/pnnl/deimos
- **NIST SRM 1950** (Multiomic reference material (human plasma extract spiked with ADH/enolase digests) used as the test mixture whose per-class errors and RMSE are compiled.)

## Examples

```
python MOCCal.exe  # run inside the extracted MOCCal folder; parse the 'Output' folder afterwards to compile per-class error percentages and RMSE values
```

## Evaluation signals

- Percent errors must match the published table: 0.02% (lipids), 0.75% (small molecules), 0.01%/0.06%/0.13% (peptides z=1/2/3).
- RMSE values must match the published table: 0.28 (lipids), 0.56 (small molecules), 0.09/0.09/0.24 (peptides z=1/2/3).
- Feature m/z constraints held: small molecules < m/z 550, lipids > m/z 300; features outside constraints should be reassigned to the next closest ΔCCS match.
- The summary CSV must contain exactly one row per biomolecular class (5 classes: lipids, small molecules, peptides z=1,2,3) with both error% and RMSE columns populated.

## Limitations

- MOCCal is currently limited to small molecule, lipid, and peptide calibrations on TWIM-MS platforms; classes outside these (e.g. negative-ionization small molecules) are not yet supported.
- Calibration biases can confound feature identification when a single power-law equation is applied instead of class-specific calibrations.
- Errors and RMSE are only reproducible if the same calibrant set (poly-dl-alanine, Waters Major Mix, Agilent ESI-L Tune Mix) and the same NIST SRM 1950 extract are used.

## Evidence

- [abstract] MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves and uses the difference (in Å2) between the calibrated TWCCSN2 value and DTCCSN2 vs m/z regression lines to determine the best calibration curve.: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves and uses the difference (in Å2) between the calibrated TWCCSN2 value and DTCCSN2 vs m/z regression"
- [results] MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters: "MOCCal will calculate up to three calibrated TWCCSN2 values using each set of class-specific calibration parameters"
- [results] Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
- [results] MOCCal utilizes biomolecule-specific TWIM CCS calibration equations: "MOCCal utilizes biomolecule-specific TWIM CCS calibration equations"
- [readme] download the MOCCal zip file and extract all, and use the MOCCal executable file. The only requirement is that the MOCCal executable is in a location that also contains a folder titled “Output.”: "download the MOCCal zip file ([MOCCalZip](https://drive.google.com/drive/folders/1emXyk9YWUuBdkcCdkMoD0RCHJLZxQZcr?usp=sharing)), extract all, and use the MOCCal executable file. The only requirement"
