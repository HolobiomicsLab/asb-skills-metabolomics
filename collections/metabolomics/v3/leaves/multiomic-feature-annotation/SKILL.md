---
name: multiomic-feature-annotation
description: Use when a traveling-wave IM-MS feature table (m/z, charge, retention
  time, arrival time) contains unidentified small-molecule, lipid, and peptide features
  from a mixed multiomic mixture and class-specific TWCCSN2 values are needed without
  prior identification.
when_to_use_negative:
- The data are already identified feature annotations (e.g. named lipids/peptides)
  — use direct CCS comparison rather than class-assignment-by-ΔCCS
- Only one biomolecular class is present and a single power law calibration suffices;
  MOCCal's multi-class machinery adds no value
- Negative ionization mode data or chemical classes outside small molecules, lipids,
  and peptides — the article notes MOCCal is currently limited to these classes and
  positive-mode datasets
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Python tool that calibrates each feature with up to three class-specific power
    law equations, assigns the biomolecular class by smallest ΔCCS to the DTCCSN2-vs-m/z
    trendline, and writes calibrated CCS values to the Output folder
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Used in the RawDT workflow to extract calibrant arrival times from raw .mzML
    files and fit them to a Gaussian distribution, recording the apex as the arrival
    time for calibration
  repo: http://github.com/pnnl/deimos
- name: Unified CCS Compendium
  role: Source of reference DTCCSN2 values used to build the small-molecule, lipid,
    and peptide DTCCSN2-vs-m/z regression trendlines against which ΔCCS is computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:ca02e891ba69e725045fec2664404c5bb6c067382b57e716a664882743c99cd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - Agilent ESI-L Tune Mix
  - DEIMoS
  - MOCCal
  - NIST SRM 1950
  - Unified CCS Compendium
  - poly-dl-alanine
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# multiomic-feature-annotation

## Summary

Biomolecular-class annotation of unknown TWIM-MS features by calibrating each arrival time with up to three class-specific power law equations (small molecule, lipid, peptide) and selecting the class whose calibrated TWCCSN2 value sits closest to its DTCCSN2-vs-m/z regression trendline.

## When to use

Use when a traveling-wave IM-MS feature table (m/z, charge, retention time, arrival time) contains unidentified small-molecule, lipid, and peptide features from a mixed multiomic mixture and class-specific TWCCSN2 values are needed without prior identification.

## When NOT to use

- The data are already identified feature annotations (e.g. named lipids/peptides) — use direct CCS comparison rather than class-assignment-by-ΔCCS
- Only one biomolecular class is present and a single power law calibration suffices; MOCCal's multi-class machinery adds no value
- Negative ionization mode data or chemical classes outside small molecules, lipids, and peptides — the article notes MOCCal is currently limited to these classes and positive-mode datasets

## Inputs

- Experimental feature spreadsheet with feature identifiers, m/z, charge state, retention time, and arrival time values (per MOCCal's UserDT template)
- Calibrant arrival-time and reference DTCCSN2 spreadsheets (e.g. poly-dl-alanine, Waters Major Mix IMS/Tof Calibration Kit, Agilent ESI-L Tune Mix, SPLASH Lipidomix)
- Optionally, raw calibration .mzML files for the RawDT/DEIMoS path
- Reference DTCCSN2 regression lines from the Unified CCS Compendium (Supporting Information Tables 1-3)

## Outputs

- Per-feature biomolecular class assignment (small molecule, lipid, or peptide with charge z = 1, 2, or 3)
- Class-specific calibrated TWCCSN2 values for every singly charged feature
- Per-class calibration error percentages and RMSE values (e.g. 0.02%, 0.75%, 0.01%, 0.06%, 0.13%; RMSE 0.28, 0.56, 0.09, 0.09, 0.24) written to the 'Output' folder
- Calibration effect score (1-4) indicating how discriminative the class assignment was

## How to apply

Build class-specific power law regressions from calibrant arrival times (extracted with DEIMoS and fit to a Gaussian, apex recorded) and reference drift tube CCS values from the Unified CCS Compendium, using calibrants such as poly-dl-alanine (peptide), Waters Major Mix / Agilent ESI-L Tune Mix, and SPLASH Lipidomix lipids. For each singly charged experimental feature, compute up to three calibrated TWCCSN2 values using the small-molecule, lipid, and peptide (z = 1, 2, 3) power law parameters. Apply the m/z constraints (<550 for small molecules, >300 for lipids); if the feature falls outside the m/z range of the class with the smallest ΔCCS, fall back to the next closest match by ΔCCS. Select the calibrated TWCCSN2 whose ΔCCS to the corresponding DTCCSN2-vs-m/z trendline is smallest; that class assignment defines the feature's annotation. In the flow-injection test on NIST SRM 1950 spiked with enolase/ADH digests, this produced per-class calibration errors of 0.02% (lipids), 0.75% (small molecules), and 0.01/0.06/0.13% (peptides z = 1/2/3) with RMSE of 0.28, 0.56, 0.09, 0.09, and 0.24.

## Related tools

- **MOCCal** (Python tool that calibrates each feature with up to three class-specific power law equations, assigns the biomolecular class by smallest ΔCCS to the DTCCSN2-vs-m/z trendline, and writes calibrated CCS values to the Output folder) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Used in the RawDT workflow to extract calibrant arrival times from raw .mzML files and fit them to a Gaussian distribution, recording the apex as the arrival time for calibration) — http://github.com/pnnl/deimos
- **Unified CCS Compendium** (Source of reference DTCCSN2 values used to build the small-molecule, lipid, and peptide DTCCSN2-vs-m/z regression trendlines against which ΔCCS is computed)

## Examples

```
Extract the MOCCal zip, place the MOCCal executable beside a folder titled 'Output', load the packaged UserDT example templates (poly-dl-alanine, Waters Major Mix, Agilent ESI-L Tune Mix calibrants plus the NIST SRM 1950 feature table spiked with ADH/enolase digests), and run the MOCCal executable; parse the 'Output' folder for per-class calibration errors and RMSE values.
```

## Evaluation signals

- Per-class calibration errors match the published values: lipids 0.02%, small molecules 0.75%, peptides 0.01% (z=1), 0.06% (z=2), 0.13% (z=3)
- Per-class RMSE values match 0.28 (lipid), 0.56 (small molecule), 0.09 (peptide z=1), 0.09 (z=2), 0.24 (z=3)
- Every feature falls within the m/z constraints of its assigned class (small molecules < m/z 550, lipids > m/z 300) or the fallback 'next closest match by ΔCCS' logic was triggered
- Correct class assignment achieved for ≥70% of lipids and peptides with calibration errors within ±1% (article's reported threshold)
- Output folder contains calibrated TWCCSN2 values and class assignments for every singly charged feature in the input spreadsheet

## Limitations

- MOCCal is currently limited to small molecule, lipid, and peptide calibrations within TWIM-MS platforms; other endogenous or xenobiotic chemical classes are not covered
- Negative ionization mode data sets are not yet calibrated; user-driven expansion is welcomed
- Multiply charged peptide assignments receive a default score of 0 for calibration effect, so discriminative power is lower for those features
- Calibration biases from a single power law equation can confound feature identification if the wrong class is forced onto a feature

## Evidence

- [abstract] calibrate every experimental arrival time with up to three class-specific calibration curves: "MOCCal calibrates every experimental arrival time with up to three class-specific calibration curves"
- [results] m/z constraints: [evidence span withheld — anchor status: not_found]
- [results] selection rule: "The calibrated TWCCSN2 value with the smallest ΔCCS to the corresponding trendline is selected as the correct match"
- [results] experimental input format: "Experimental data can be imported as a spreadsheet containing feature identifiers along with their corresponding m/z, charge state, retention time, and arrival time values"
- [results] calibrant extraction workflow: [evidence span withheld — anchor status: not_found]
- [discussion] scope limitation: "MOCCal around small molecule, lipid, and peptide calibrations with the intention of enabling mixed multiomic analyses on TWIM-MS platforms"
- [results] accuracy benchmark: "MOCCal provides the best assignment results for lipids and singly charged peptides when the calibration errors are ±0.5% but still achieves the correct assignment for ≥70% of lipids and peptides with"
- [readme] tool requirement: "If you plan to use raw calibration files, you will need to install pnnl's DEIMoS, version 1.3.2"
