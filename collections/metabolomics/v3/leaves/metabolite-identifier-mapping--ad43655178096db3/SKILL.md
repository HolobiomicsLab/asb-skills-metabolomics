---
name: metabolite-identifier-mapping
description: Use when annotating preprocessed MSI feature lists (measured m/z values)
  with metabolite names and adduct forms using an organ-specific metabolite database,
  e.g. mapping brain/liver/kidney/heart/spleen/lung/muscle/pancreas/serum metabolites
  from AFADESI-MSI mouse body data.
when_to_use_negative:
- Input is raw, unprocessed MSI data rather than a preprocessed dataset with m/z features
  in the first column — MSIannotator expects preprocessed MSIdata.
- The task is targeted LC-MS/MS metabolite quantification without spatial context
  — this skill is specific to MSI annotation workflows using organ-specific databases.
- The metabolite database and MSIdata detection modes (positive/negative) or organ
  sources are mismatched — annotation would produce spurious matches.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MSIannotator
  role: Automated MATLAB annotation tool that maps metabolites and adduct ions in
    MSI datasets against a supplied metabolite database.
  repo: github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which MSIannotator.m is executed; MSIdata and Database
    sheets are imported as string arrays and matched via m/z error filtering.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:ac6ae875617da7c1c77a42f8db3f240bcac06807039215fe6fe4ea2d4300800d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-identifier-mapping

## Summary

Annotate metabolites and adduct forms in preprocessed mass spectrometry imaging (MSI) datasets by matching measured m/z values against an organ-specific metabolite database, enabling organ-specific spatial metabolite distribution analysis in whole-body mouse MSI.

## When to use

Use when annotating preprocessed MSI feature lists (measured m/z values) with metabolite names and adduct forms using an organ-specific metabolite database, e.g. mapping brain/liver/kidney/heart/spleen/lung/muscle/pancreas/serum metabolites from AFADESI-MSI mouse body data.

## When NOT to use

- Input is raw, unprocessed MSI data rather than a preprocessed dataset with m/z features in the first column — MSIannotator expects preprocessed MSIdata.
- The task is targeted LC-MS/MS metabolite quantification without spatial context — this skill is specific to MSI annotation workflows using organ-specific databases.
- The metabolite database and MSIdata detection modes (positive/negative) or organ sources are mismatched — annotation would produce spurious matches.

## Inputs

- MSIdata.xlsx — preprocessed MSI dataset with measured m/z in the first column (e.g. liver tissue MSI data in positive mode)
- Organ specific metabolite database.xlsx — organ-specific metabolite database: 1298 metabolites in positive mode (Sheet1), 515 in negative mode (Sheet2), divided by brain (Sheet3) through serum (Sheet11)
- Database.xlsx — example metabolite database for a specific organ and ionization mode

## Outputs

- Annotator file — annotated metabolite list with measured m/z in the first column and intermediate plus ultimate annotated metabolite names with their adduct forms in subsequent columns

## How to apply

Import two sheets into MATLAB as string arrays named 'MSIdata' (preprocessed MSI dataset with measured m/z in the first column) and 'Database' (organ-specific metabolite database with metabolite names in the first column and theoretical m/z in adduct-form columns). Verify that the selected organ database (brain=Sheet3, liver=Sheet4, kidney=Sheet5, heart=Sheet6, spleen=Sheet7, lung=Sheet8, muscle=Sheet9, pancreas=Sheet10, serum=Sheet11) and detection mode (positive: 1298 metabolites; negative: 515 metabolites) are consistent with the MSIdata being annotated. Run MSIannotator.m, which computes abs(theor-mea)/theor with a default 10 ppm m/z error threshold (modifiable at line 25, e.g. to 5 ppm by replacing '10·10^(-6)' with '5·10^(-6)'). The script generates an 'Annotator' output file containing intermediate and ultimate annotated metabolite names with their adduct forms.

## Related tools

- **MSIannotator** (Automated MATLAB annotation tool that maps metabolites and adduct ions in MSI datasets against a supplied metabolite database.) — github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which MSIannotator.m is executed; MSIdata and Database sheets are imported as string arrays and matched via m/z error filtering.)

## Examples

```
MSIdata = readmatrix('MSIdata.xlsx'); Database = readmatrix('Database.xlsx'); MSIannotator  % run MSIannotator.m with 10 ppm default: abs(theor-mea)/theor<10*10^(-6)
```

## Evaluation signals

- Confirmed m/z error criterion holds for every annotation: abs(theor-mea)/theor < 10·10^(-6) (or the user-set threshold at line 25).
- Detection mode consistency: positive-mode MSIdata annotated against the 1298-metabolite positive-mode database, negative-mode against the 515-metabolite negative-mode database.
- Organ database consistency: the organ sheet (e.g. liver=Sheet4) matches the organ/tissue origin of the MSIdata being annotated.
- Output 'Annotator' file contains the same number of rows (measured m/z features) as MSIdata's first column, with non-empty intermediate and ultimate metabolite annotations plus adduct forms.
- Downstream organ-specificity check: per-organ metabolite membership sets derived from annotated results should reproduce the reported finding that metabolites distribute across organs in an organ-specific manner conserved across mouse strains.

## Limitations

- m/z error threshold defaults to 10 ppm but must be manually modified in the code (line 25) if a stricter or looser tolerance is desired; no CLI parameter exists.
- No changelog found for the source repository, so version history and behavioral changes are not trackable.
- Organ/detection-mode mismatch between Database and MSIdata is only detectable by manual pre-check — the tool does not automatically validate this consistency.
- Annotation relies on adduct-form differences between LC-MS and AFADESI-MSI, so adduct assignments may differ between the two acquisition techniques.

## Evidence
