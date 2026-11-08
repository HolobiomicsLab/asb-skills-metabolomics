---
name: calibration-error-scoring
description: Use when a TWIM-MS feature has been calibrated with multiple class-specific
  power law curves (small molecule, lipid, peptide) and you must record how confident/sensitive
  the class assignment and resulting CCS value are.
when_to_use_negative:
- Feature is multiply charged — MOCCal assigns a default score of 0 because class
  arbitration across the three calibrations is not performed
- Only one class-specific calibration candidate exists for the feature, so no 'next
  closest match' is available to compare against
- Data are in negative ionization mode or belong to chemical classes outside small
  molecule/lipid/peptide scope — the article notes MOCCal is not yet extended to these
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MOCCal
  role: Computes the calibrated TWCCSN2 values per class, assigns the class by smallest
    ΔCCS, and emits the 1–4 calibration effect score for each feature
  repo: https://github.com/HinesLab/MOCCal
- name: DEIMoS
  role: Extracts and Gaussian-fits arrival time data from raw .mzML calibration files
    (version 1.3.2) to supply the arrival times that feed the calibration and scoring
    pipeline
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04290
    title: 'MOCCal: A Multiomic CCS Calibrator for Traveling Wave Ion Mobility Mass
      Spectrometry'
  source_content_hash: sha256:cea7e0ac9b95c625b2e74a4c8d158558d3e97cd4277b36ac364b36fe27d01db7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HinesLab/MOCCal
  tools:
  - DEIMoS
  - MOCCal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# calibration-error-scoring

## Summary

Scoring scheme from MOCCal that quantifies how consequential a biomolecular-class assignment decision was during TWIM-MS CCS calibration, by comparing the calibrated TWCCSN2 value from the assigned class's power law against the next closest candidate class. Scores 1–4 flag when class choice materially affects the reported CCS value; multiply charged features default to 0.

## When to use

Use when a TWIM-MS feature has been calibrated with multiple class-specific power law curves (small molecule, lipid, peptide) and you must record how confident/sensitive the class assignment and resulting CCS value are.

## When NOT to use

- Feature is multiply charged — MOCCal assigns a default score of 0 because class arbitration across the three calibrations is not performed
- Only one class-specific calibration candidate exists for the feature, so no 'next closest match' is available to compare against
- Data are in negative ionization mode or belong to chemical classes outside small molecule/lipid/peptide scope — the article notes MOCCal is not yet extended to these

## Inputs

- Per-feature calibrated TWCCSN2 values from each candidate class-specific power law calibration (small molecule, lipid, singly charged peptide)
- Per-feature assigned biomolecular class (smallest ΔCCS match subject to m/z constraints: small molecules <m/z 550, lipids >m/z 300)
- Experimental feature spreadsheet (feature identifiers, m/z, charge state, retention time, arrival time)

## Outputs

- Per-feature assignment score (0–4) quantifying the percent difference between the assigned-class calibration and the next closest match
- Enriched feature_class_assignments.csv rows containing class label, calibrated TWCCSN2 value, ΔCCS, and score

## How to apply

After MOCCal calibrates a feature's arrival time with up to three class-specific power law equations and assigns the class with the smallest ΔCCS to its DTCCSN2 vs m/z trendline, compute the percent difference between the calibrated TWCCSN2 value derived from the assigned class and the next closest candidate. Bin that difference into the MOCCal scale: score 1 = <1%, score 2 = 1–3%, score 3 = 3–6%, score 4 = ≥6%. Record this score alongside the calibrated TWCCSN2 value and ΔCCS in the per-feature assignment table (e.g., feature_class_assignments.csv) so downstream users know when the reported CCS value depends critically on correct class assignment. Assign the default score 0 to multiply charged features, which are not arbitrated across classes. The score is informative rather than corrective — a high score signals a feature whose identification would benefit from orthogonal evidence (exact mass, MS/MS, LipidPioneer/CCS Compendium lookup).

## Related tools

- **MOCCal** (Computes the calibrated TWCCSN2 values per class, assigns the class by smallest ΔCCS, and emits the 1–4 calibration effect score for each feature) — https://github.com/HinesLab/MOCCal
- **DEIMoS** (Extracts and Gaussian-fits arrival time data from raw .mzML calibration files (version 1.3.2) to supply the arrival times that feed the calibration and scoring pipeline) — http://github.com/pnnl/deimos

## Examples

```
python MOCCal_RawDT.py  # run in the DEIMoS 1.3.2 virtual environment; executable variant requires a sibling folder titled 'Output'
```

## Evaluation signals

- Every scored feature carries an integer score in the 0–4 range, with 0 reserved for multiply charged assignments
- Score 1 features should show <1% difference between the assigned-class calibrated TWCCSN2 value and the next closest candidate; score 4 features ≥6%
- Scores are internally consistent with reported ΔCCS values — e.g., the m/z 692.517 lipid case had 0.18% error from its own class calibration vs 1.8% and 2.6% from small molecule/peptide curves
- Assignment accuracy over the scored feature set falls within the reported MOCCal envelope: ≥70% correct assignment for lipids and singly charged peptides with calibration errors of ±0.5–1%

## Limitations

- Scores reflect agreement between candidate calibrations, not absolute correctness — a low score can still accompany a misassignment if two class curves give similar (wrong) values
- Multiply charged features receive a default score of 0, so the score carries no confidence information for them
- MOCCal is currently scoped to small molecule, lipid, and peptide calibrations on TWIM-MS platforms and is not yet extended to negative ionization mode or other chemical classes

## Evidence

- [results] score thresholds: "A score of 4 indicates that there is a difference of 6% or greater between the calibrated CCS value derived from the assigned class and the next closest match."
- [readme] readme scope: "MOCCal offers class assignment and CCS calculations without need for identifying the features first."
- [intro] workflow context: "The proposed workflow involves the combination of CCS calibration parameter determination, biomolecular class assignment of unknown features, and the application of structurally matched CCS"
