---
name: organ-specific-metabolite-database-selection
description: Use when preparing to annotate a preprocessed ambient MSI dataset (e.g.
  AFADESI-MSI mouse tissue data) and you must choose which database sheet — organ
  and positive/negative mode — to supply as the 'Database' input before running MSIannotator.m.
when_to_use_negative:
- The MSI dataset comes from an organ or tissue type not covered by the organ-specific
  database sheets (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum)
  or from a species other than mouse, where a user-supplied database must be built
  instead.
- The detection mode of the available database sheet (positive or negative) does not
  match the ionization mode of the MSI data to be annotated — supplying the wrong
  mode would compare measured m/z against theoretical adduct ions that were not formed.
- The input is raw, unprocessed mass spectrometry imaging data rather than a preprocessed
  MSI dataset with measured m/z in the first column; database selection is a pre-annotation
  step, not a preprocessing step.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MSIannotator
  role: Automated annotation tool that consumes the organ- and mode-selected Database
    to annotate metabolites and adducts in the MSI dataset
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: MATLAB script that performs the annotation using the selected Database against
    MSIdata with the 10 ppm default m/z error threshold
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Environment in which the database sheet is imported as a string array named
    Database and the annotation script is executed
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:ac9b8fe0f2d9b541c0fe4385482433d8f476e4767816806545301f78259d648a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# organ-specific-metabolite-database-selection

## Summary

Selects and validates an organ- and ionization-mode-specific metabolite database (e.g. the MSIannotator 'Organ specific metabolite database.xlsx' sheets, 1298 metabolites in positive mode and 515 in negative mode, split by brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum) so that downstream m/z annotation is matched against the correct theoretical adduct ions for the tissue being imaged.

## When to use

Use when preparing to annotate a preprocessed ambient MSI dataset (e.g. AFADESI-MSI mouse tissue data) and you must choose which database sheet — organ and positive/negative mode — to supply as the 'Database' input before running MSIannotator.m.

## When NOT to use

- The MSI dataset comes from an organ or tissue type not covered by the organ-specific database sheets (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum) or from a species other than mouse, where a user-supplied database must be built instead.
- The detection mode of the available database sheet (positive or negative) does not match the ionization mode of the MSI data to be annotated — supplying the wrong mode would compare measured m/z against theoretical adduct ions that were not formed.
- The input is raw, unprocessed mass spectrometry imaging data rather than a preprocessed MSI dataset with measured m/z in the first column; database selection is a pre-annotation step, not a preprocessing step.

## Inputs

- Preprocessed MSI dataset (e.g. MSIdata.xlsx, MSI data of liver tissue in positive mode) with measured m/z in the first column
- Organ specific metabolite database.xlsx containing 1298 metabolites in positive mode (Sheet1) and 515 metabolites in negative mode (Sheet2), subdivided into organ-specific sheets (Sheet3 brain, Sheet4 liver, Sheet5 kidney, Sheet6 heart, Sheet7 spleen, Sheet8 lung, Sheet9 muscle, Sheet10 pancreas, Sheet11 serum)
- Knowledge of the organ imaged and the ionization mode (positive or negative) used to acquire the MSI data

## Outputs

- A validated 'Database' string array in MATLAB whose organ and detection mode match the MSIdata to be annotated, ready as input to MSIannotator.m
- Documented consistency check confirming the selected organ database and detection mode match the MSIdata before annotation

## How to apply

Import the candidate database sheet from 'Organ specific metabolite database.xlsx' into MATLAB as a string array named 'Database' alongside the preprocessed MSI dataset named 'MSIdata' (which must contain measured m/z values in its first column). Before annotation, explicitly check that the selected organ (e.g. liver, brain, kidney, heart, spleen, lung, muscle, pancreas, or serum) and the detection mode (positive or negative) of the Database are consistent with the tissue and mode of the MSIdata being annotated — for example, use Sheet4 (liver, positive mode) for liver tissue acquired in positive mode. Each database entry pairs a metabolite name in the first column with theoretical m/z values in specified adduct-form columns, covering the adduct species actually observed in AFADESI-MSI (which can differ from LC-MS adducts). Only after this consistency check should MSIannotator.m be run with its default 10 ppm m/z error criterion (line 25: if abs(theor-mea)/theor < 10*10^(-6)), which users may tighten to e.g. 5 ppm by editing that line. The rationale is that the authors established a metabolite database for MSI consisting of 27,407 adduct ions from 1620 unique metabolites identified across eight mouse organs and serum, organized organ-specifically so that annotation reflects the metabolites actually present in the tissue rather than a generic whole-body list.

## Related tools

- **MSIannotator** (Automated annotation tool that consumes the organ- and mode-selected Database to annotate metabolites and adducts in the MSI dataset) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator.m** (MATLAB script that performs the annotation using the selected Database against MSIdata with the 10 ppm default m/z error threshold) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Environment in which the database sheet is imported as a string array named Database and the annotation script is executed)

## Examples

```
Import 'Organ specific metabolite database.xlsx' Sheet4 into MATLAB as string array Database; rename MSIdata.xlsx sheet as MSIdata; run MSIannotator.m (default 10 ppm).
```

## Evaluation signals

- Confirm the selected Database sheet's organ and detection mode exactly match the tissue and ionization mode of MSIdata before running MSIannotator.m — e.g. Sheet4 (liver, positive mode) for liver tissue acquired in positive mode.
- Verify that every entry in the chosen Database has a metabolite name in the first column and theoretical m/z values in specified adduct-form columns, matching the required input file structure.
- After annotation, check that the generated Annotator output lists measured m/z values from MSIdata in the first column and that all reported metabolite/adduct assignments satisfy the configured m/z error criterion (default 10 ppm: abs(theor-mea)/theor < 10*10^(-6)).
- If a tighter tolerance is set (e.g. 5 ppm via line 25 modification), re-run annotation and confirm the annotated metabolite list does not change unexpectedly beyond the expected reduction in marginal matches.
- Cross-check that the number of theoretical adduct ions available in the selected organ sheet is consistent with the reported database scale (27,407 adduct ions from 1620 unique metabolites across eight organs and serum).

## Limitations

- The organ-specific database is mouse-based (eight mouse organs plus serum); applying it to other species or tissues outside this set requires a user-supplied database.
- The database covers only the adduct forms evaluated for AFADESI-MSI; adduct distributions in other MSI ion sources may differ from those in LC-MS.
- The default 10 ppm m/z error range is generous and may produce ambiguous annotations; users must explicitly edit line 25 (e.g. to 5 ppm) if a stricter criterion is needed, and no changelog is available for the source repository.

## Evidence
