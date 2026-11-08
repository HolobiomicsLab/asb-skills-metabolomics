---
name: pathway-metabolite-mapping
description: Use when annotating m/z features from preprocessed organ-specific MSI
  data (e.g. AFADESI-MSI in positive/negative mode) and tracing which metabolic pathways
  are altered across multiple organs, such as liver, brain, and kidney.
when_to_use_negative:
- 'The input is unprocessed or non-imaging data: MSIannotator expects a preprocessed
  MSI dataset with measured m/z in the first column, not raw spectral files.'
- The database organ or detection mode does not match the MSI data being annotated
  (e.g. a liver positive-mode database paired with brain negative-mode MSIdata).
- 'The goal is LC-MS/MS compound identification rather than MSI metabolite annotation:
  the workflow was built around the adduct form differences between LC-MS and AFADESI-MSI.'
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: MSIannotator
  role: Automated MATLAB tool that annotates metabolites and adducts in the MSI dataset
    against the organ-specific database, producing the Annotator output used for pathway
    mapping
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which MSIdata and Database sheets are loaded as string
    arrays and MSIannotator.m is executed
- name: MSIannotator.m
  role: Script that matches measured m/z against theoretical adduct m/z with a default
    10 ppm error range (modifiable at line 25)
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:340b5f8ddec4e2b47a837b19eb8ac34eb6208babea4dcc10545554cc405c4338
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pathway-metabolite-mapping

## Summary

Maps organ-specific metabolite annotations from ambient mass spectrometry imaging (AFADESI-MSI) data onto shared metabolic pathways to reveal spatial metabolic alterations across organs, e.g. identifying the alanine, aspartate, and glutamate metabolism pathway as simultaneously altered in the liver, brain, and kidney of diabetic mice.

## When to use

Use when annotating m/z features from preprocessed organ-specific MSI data (e.g. AFADESI-MSI in positive/negative mode) and tracing which metabolic pathways are altered across multiple organs, such as liver, brain, and kidney.

## When NOT to use

- The input is unprocessed or non-imaging data: MSIannotator expects a preprocessed MSI dataset with measured m/z in the first column, not raw spectral files.
- The database organ or detection mode does not match the MSI data being annotated (e.g. a liver positive-mode database paired with brain negative-mode MSIdata).
- The goal is LC-MS/MS compound identification rather than MSI metabolite annotation: the workflow was built around the adduct form differences between LC-MS and AFADESI-MSI.

## Inputs

- Preprocessed MSI dataset sheet (e.g. MSIdata.xlsx; measured m/z in the first column), imported into MATLAB as a string array named MSIdata
- Organ-specific metabolite database sheet (Organ specific metabolite database.xlsx: 1298 metabolites positive mode, 515 negative mode; sheets divided by brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum), imported as a string array named Database

## Outputs

- Annotator.csv with measured m/z in the first column followed by intermediate and ultimate annotated metabolite names and their adduct forms
- Pathway-filtered metabolite list with organ assignments (e.g. alanine/aspartate/glutamate metabolism metabolites annotated in liver, brain, kidney)

## How to apply

Import the preprocessed MSI dataset sheet (e.g. MSIdata.xlsx, liver tissue in positive mode) and the matching organ/mode sheet from Organ specific metabolite database.xlsx into MATLAB as string arrays named MSIdata and Database, verifying the organ and detection mode are consistent between the two. Run MSIannotator.m, which matches measured m/z in the first column of MSIdata against theoretical adduct m/z values using the default 10 ppm error criterion (abs(theor-mea)/theor < 10*10^(-6), line 25, modifiable to e.g. 5 ppm). Save the output as Annotator.csv, containing measured m/z followed by intermediate and ultimate annotated metabolite names and their adduct forms. Then filter the annotations for metabolites belonging to a target pathway (here the alanine, aspartate, and glutamate metabolism pathway) and record each metabolite's organ assignment to determine which organs show spatial alterations and whether a single pathway is simultaneously perturbed across them, as reported for the liver, brain, and kidney in diabetic mice.

## Related tools

- **MSIannotator** (Automated MATLAB tool that annotates metabolites and adducts in the MSI dataset against the organ-specific database, producing the Annotator output used for pathway mapping) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which MSIdata and Database sheets are loaded as string arrays and MSIannotator.m is executed)
- **MSIannotator.m** (Script that matches measured m/z against theoretical adduct m/z with a default 10 ppm error range (modifiable at line 25)) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
run MSIannotator.m  % with MSIdata and Database string arrays loaded in MATLAB; default 10 ppm m/z error at line 25: abs(theor-mea)/theor<10*10^(-6)
```

## Evaluation signals

- Every annotation in Annotator.csv satisfies the m/z error criterion abs(theor-mea)/theor < 10*10^(-6) (or the user-modified ppm threshold at line 25).
- Annotator output schema check: first column contains measured m/z, followed by intermediate and ultimate annotated metabolite names and their adduct forms.
- Consistency check passed: the selected organ and detection mode of the Database match those of the MSIdata being annotated before running MSIannotator.m.
- Pathway mapping reproduces the reported pattern: alanine, aspartate, and glutamate metabolism pathway metabolites are annotated in all three organs (liver, brain, kidney) of diabetic mice.

## Limitations

- Annotation depends on the supplied database (27,407 adduct ions for MSI across 1298 positive-mode and 515 negative-mode metabolites); metabolites absent from the organ database cannot be annotated.
- The 10 ppm m/z error threshold is a default that users must consciously adjust (line 25) for stricter or looser matching; incorrect thresholds change annotation confidence.
- The database is organized by organ and detection mode, so using a mismatched sheet (wrong organ or positive/negative mode) yields invalid annotations.

## Evidence
