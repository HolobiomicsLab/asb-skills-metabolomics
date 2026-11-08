---
name: matlab-script-execution
description: Use when preprocessed MSI data (e.g. AFADESI-MSI of mouse liver in positive
  mode) needs metabolite and adduct annotation via the MATLAB script MSIannotator.m
  against an organ-specific database.
when_to_use_negative:
- Input is raw, unprocessed MSI data — the tool is designed to annotate 'processed
  MSI data', so preprocessing (m/z feature extraction) must be done first
- The supplied Database's organ or detection mode does not match the MSIdata to be
  annotated (e.g. a liver positive-mode database against kidney negative-mode data)
- The analysis requires LC-MS/MS-level metabolite identification rather than MSI adduct-form
  annotation — the database construction accounts for adduct form differences between
  LC-MS and AFADESI-MSI, so LC-MS annotation conventions do not transfer directly
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MSIannotator.m
  role: MATLAB script executed to annotate the features in MSIdata with the Database,
    using a line-25 m/z error criterion (default 10 ppm)
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which the MSIdata and Database string arrays are imported
    and MSIannotator.m is carried out
- name: MSIannotator
  role: The automated annotation tool whose script implements metabolite and adduct
    annotation for the MSI dataset
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:d751b8dabaed7875c909dde15c481633b6670b35a17ddf6e6a2af44d1dc31445
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# matlab-script-execution

## Summary

Executes the MSIannotator.m MATLAB script to annotate metabolites and their adduct forms in preprocessed mass spectrometry imaging (MSI) data by matching measured m/z against an organ-specific metabolite database within a default 10 ppm error tolerance.

## When to use

Use when preprocessed MSI data (e.g. AFADESI-MSI of mouse liver in positive mode) needs metabolite and adduct annotation via the MATLAB script MSIannotator.m against an organ-specific database.

## When NOT to use

- Input is raw, unprocessed MSI data — the tool is designed to annotate 'processed MSI data', so preprocessing (m/z feature extraction) must be done first
- The supplied Database's organ or detection mode does not match the MSIdata to be annotated (e.g. a liver positive-mode database against kidney negative-mode data)
- The analysis requires LC-MS/MS-level metabolite identification rather than MSI adduct-form annotation — the database construction accounts for adduct form differences between LC-MS and AFADESI-MSI, so LC-MS annotation conventions do not transfer directly

## Inputs

- MSIdata — preprocessed MSI dataset as a MATLAB string array, measured m/z in the first column (example: MSIdata.xlsx, liver tissue MSI data in positive mode)
- Database — metabolite database as a MATLAB string array, metabolite names in the first column and theoretical m/z in specified adduct-form columns (example: Database.xlsx, liver metabolite database in positive mode)
- Organ specific metabolite database.xlsx — established database with 1298 metabolites in positive mode (Sheet1), 515 in negative mode (Sheet2), divided by organ into Sheet3 (brain) through Sheet11 (serum)

## Outputs

- Annotator — annotated metabolite list with measured m/z in the first column, intermediate and ultimate annotated metabolite names and their adduct forms in the following columns (saveable as Annotator.xlsx)

## How to apply

Import two sheets into MATLAB as string arrays named MSIdata (preprocessed MSI dataset with measured m/z in the first column) and Database (metabolite names in the first column, theoretical m/z in adduct-form columns). Verify that the selected organ (e.g. liver) and detection mode (positive or negative) of the Database are consistent with the MSIdata to be annotated. Run MSIannotator.m, whose default m/z error range is 10 ppm enforced at line 25 as abs(theor-mea)/theor < 10*10^(-6); modify line 25 (e.g. to 5*10^(-6) for 5 ppm) only if a tighter tolerance is needed. The script writes a new file 'Annotator' listing measured m/z in the first column followed by intermediate and ultimate annotated metabolite names and their adduct forms, which the user can copy or save (e.g. as Annotator.xlsx).

## Related tools

- **MSIannotator.m** (MATLAB script executed to annotate the features in MSIdata with the Database, using a line-25 m/z error criterion (default 10 ppm)) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which the MSIdata and Database string arrays are imported and MSIannotator.m is carried out)
- **MSIannotator** (The automated annotation tool whose script implements metabolite and adduct annotation for the MSI dataset) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
MSIdata = readtable('MSIdata.xlsx'); Database = readtable('Database.xlsx'); run('MSIannotator.m')  % line 25: if abs(theor-mea)/theor<10*10^(-6)
```

## Evaluation signals

- The output file 'Annotator' exists and is non-empty after script execution
- First column of the Annotator output contains the measured m/z values from MSIdata, followed by annotated metabolite names and adduct forms
- Each annotated row's adduct ion theoretical m/z matches its measured m/z within the 10 ppm criterion, i.e. abs(theor-mea)/theor < 10*10^(-6)
- The Database sheet used (organ and mode, e.g. Sheet4 liver positive mode) matches the organ and detection mode of MSIdata before the run
- Changing line 25's threshold (e.g. 10*10^(-6) to 5*10^(-6)) predictably changes the number of annotations, confirming the tolerance is the active filter

## Limitations

- The default m/z error range of 10 ppm is hardcoded at line 25 and must be manually edited in the script if a different tolerance is required
- Annotation confidence depends on the organ-specific database coverage (1298 metabolites positive mode / 515 negative mode across 8 organs plus serum); metabolites outside the database cannot be annotated
- No changelog was found for the source repository, so version drift of the script cannot be tracked from the repo alone

## Evidence
