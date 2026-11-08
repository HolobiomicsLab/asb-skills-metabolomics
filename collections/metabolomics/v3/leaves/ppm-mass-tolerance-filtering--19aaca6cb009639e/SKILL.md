---
name: ppm-mass-tolerance-filtering
description: Use when annotating precursor m/z values from an MS-DIAL peak list (e.g.
  CSH-Q Exactive HF positive-ion lipidomics) against an in-silico library such as
  LipidBlast and you need a fixed mass-tolerance window to accept matches as MSI level
  3 annotations.
when_to_use_negative:
- The precursor already has an MS/MS match at MSI level 1 or 2 (authentic standard
  or precursor + MS/MS matching) — accurate-mass-only lookup would lower confidence.
- The instrument mass accuracy is unknown or substantially worse than the 5 mDa window
  — filtering at a fixed tolerance without verified calibration can produce false
  annotations.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CSH-Q Exactive HF mass spectrometry
  role: Generates the precursor accurate-mass peak list that the mass-tolerance filter
    is applied to; its mass accuracy defines the feasible tolerance window
- name: MS-DIAL
  role: Produces the lipidomics output peak list whose precursor m/z values are filtered
    with the 5 mDa LipidBlast tolerance
- name: LipidBlast v49 m/z lookup macro
  role: Performs the accurate-mass lookup against the in-silico lipid MS/MS library
    under the specified tolerance
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:cd5901566b0541050418efd54a0a4678c146a65db59af7e93a4c14946b0d3ac6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - LipidBlast v49 m/z lookup macro
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ppm-mass-tolerance-filtering

## Summary

Applies precursor accurate-mass tolerance filters (millidalton or ppm windows, e.g. 5 mDa for LipidBlast v49 lipid library matching) to accept MSI level 2/3 structure annotations in the CSH-Q Exactive HF urinary lipidomics workflow, using the LipidBlast v49 m/z lookup macro for MS-DIAL precursor masses.

## When to use

Use when annotating precursor m/z values from an MS-DIAL peak list (e.g. CSH-Q Exactive HF positive-ion lipidomics) against an in-silico library such as LipidBlast and you need a fixed mass-tolerance window to accept matches as MSI level 3 annotations.

## When NOT to use

- The precursor already has an MS/MS match at MSI level 1 or 2 (authentic standard or precursor + MS/MS matching) — accurate-mass-only lookup would lower confidence.
- The instrument mass accuracy is unknown or substantially worse than the 5 mDa window — filtering at a fixed tolerance without verified calibration can produce false annotations.

## Inputs

- MS-DIAL lipidomics peak list with precursor m/z values from the CSH-Q Exactive HF assay (positive ionization mode)
- LipidBlast v49 in-silico lipid MS/MS library
- precursor mass tolerance setting (5 mDa for the lipid library search)

## Outputs

- CSV annotation table containing compound name, formula, precursor m/z, and mass error (e.g. lipid_blast_annotations.csv)
- count of accepted lipid annotations (96 additional MSI level 3 hits for this dataset)

## How to apply

Read the MS-DIAL lipidomics output for the CSH-Q Exactive assay and retain precursor m/z values acquired in positive ionization mode. Convert each precursor mass and run the LipidBlast v49 m/z lookup macro against the in-silico lipid MS/MS library. Filter candidate matches using the 5 mDa precursor mass tolerance specified for the lipid library search, compiling accepted hits into a single annotation table. This precursor accurate-mass lookup alone produces MSI level 3 annotations, complementary to the MSI level 2 annotations obtained by precursor mass plus MS/MS matching. Record compound name, formula, precursor m/z, and mass error for each accepted annotation so downstream ClassyFire classification can operate on the validated hits.

## Related tools

- **CSH-Q Exactive HF mass spectrometry** (Generates the precursor accurate-mass peak list that the mass-tolerance filter is applied to; its mass accuracy defines the feasible tolerance window)
- **MS-DIAL** (Produces the lipidomics output peak list whose precursor m/z values are filtered with the 5 mDa LipidBlast tolerance)
- **LipidBlast v49 m/z lookup macro** (Performs the accurate-mass lookup against the in-silico lipid MS/MS library under the specified tolerance)

## Examples

```
lipid_blast_lookup --input msdial_peaks.csv --library lipidblast_v49 --tolerance 5mDa --ion-mode positive --output lipid_blast_annotations.csv
```

## Evaluation signals

- Count of accepted lipid annotations equals 96 for the CSH-Q Exactive urinary lipidomics dataset
- Every row in the output table includes compound name, formula, precursor m/z, and mass error
- All accepted matches fall within the 5 mDa precursor mass tolerance used for the lipid library search
- Annotations are distinguishable from MSI level 2 (precursor + MS/MS matching) hits — the table contains only accurate-mass-based (level 3) additions

## Limitations

- Precursor accurate-mass lookup alone yields MSI level 3 annotations — lower confidence than MSI level 2 (precursor + MS/MS matching) or level 1 (authentic standard) identifications
- Only 96 additional lipid annotations were obtained for the urinary lipidomics dataset at 5 mDa tolerance; tolerance choice directly affects yield and false-match risk
- The field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments in metabolomics MS/MS

## Evidence

- [methods] 5 mDa mass tolerance on positive-ionization MS-DIAL precursor masses yielded 96 additional lipid annotations: [evidence span withheld — anchor status: not_found]
- [methods] precursor accurate mass lookup alone (MSI level 3): [evidence span withheld — anchor status: not_found]
- [methods] Filter matches using the 5 mDa precursor mass tolerance: [evidence span withheld — anchor status: not_found]
