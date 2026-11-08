---
name: adduct-ion-mz-matching
description: Use when you have a preprocessed MSI feature list (measured m/z in the
  first column) and need to assign metabolite names and adduct forms, e.g. AFADESI-MSI
  liver tissue data in positive mode.
when_to_use_negative:
- The input is raw, unprocessed MSI data rather than a preprocessed feature table
  — MSIannotator explicitly annotates processed MSI data.
- The available database's organ or detection mode does not match the MSIdata (e.g.
  a positive-mode liver database paired with negative-mode brain data).
- The goal is de novo metabolite identification from MS/MS spectra rather than accurate-mass
  adduct matching against a curated database.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MSIannotator.m
  role: MATLAB script executed to match measured m/z against theoretical adduct m/z
    from the Database within the ppm error threshold
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which MSIdata and Database are loaded as string arrays
    and MSIannotator.m is carried out
- name: MSIannotator
  role: The automated annotation tool that conducts metabolite and adduct annotation
    in the MSI dataset with high efficiency and confidence
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:7791703b4f6f61dcc8dd7bccadf4f25359aea0b9d6276a5700da1378ec5aebb6
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# adduct-ion-mz-matching

## Summary

Annotates measured m/z features in preprocessed mass spectrometry imaging (MSI) data by matching them against theoretical adduct m/z values from an organ-specific metabolite database using MSIannotator.m in MATLAB, with a default 10 ppm m/z error tolerance.

## When to use

Use when you have a preprocessed MSI feature list (measured m/z in the first column) and need to assign metabolite names and adduct forms, e.g. AFADESI-MSI liver tissue data in positive mode.

## When NOT to use

- The input is raw, unprocessed MSI data rather than a preprocessed feature table — MSIannotator explicitly annotates processed MSI data.
- The available database's organ or detection mode does not match the MSIdata (e.g. a positive-mode liver database paired with negative-mode brain data).
- The goal is de novo metabolite identification from MS/MS spectra rather than accurate-mass adduct matching against a curated database.

## Inputs

- MSIdata - preprocessed MSI dataset string array with measured m/z in the first column (e.g. MSIdata.xlsx, liver tissue, positive mode)
- Database - metabolite database string array with metabolite names in the first column and theoretical m/z in adduct form columns (e.g. Database.xlsx, liver positive-mode sheet from Organ specific metabolite database.xlsx covering 1298 positive-mode / 515 negative-mode metabolites)

## Outputs

- Annotator - annotated metabolite list with measured m/z in the first column, intermediate and ultimate annotated metabolite names and their adduct forms in following columns (saved as Annotator.xlsx or Annotator.csv)

## How to apply

Import the preprocessed MSI dataset and the organ/metabolite database sheet into MATLAB as string arrays renamed MSIdata and Database, where MSIdata holds measured m/z in its first column and Database holds metabolite names plus theoretical m/z in specified adduct form columns. Verify the database's organ and detection mode (e.g. liver, positive) match the MSIdata before annotating. Run MSIannotator.m, which applies the default threshold abs(theor-mea)/theor < 10*10^(-6) at line 25; modify that line (e.g. to 5*10^(-6)) if a narrower m/z error range is needed. Save the generated Annotator output (measured m/z followed by intermediate and ultimate annotated metabolite names and adduct forms) as Annotator.xlsx or .csv.

## Related tools

- **MSIannotator.m** (MATLAB script executed to match measured m/z against theoretical adduct m/z from the Database within the ppm error threshold) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which MSIdata and Database are loaded as string arrays and MSIannotator.m is carried out)
- **MSIannotator** (The automated annotation tool that conducts metabolite and adduct annotation in the MSI dataset with high efficiency and confidence) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
MSIdata = readtable('MSIdata.xlsx'); Database = readtable('Database.xlsx'); run('MSIannotator.m'); % default 10 ppm: abs(theor-mea)/theor < 10*10^(-6) at line 25
```

## Evaluation signals

- Every annotation in the Annotator output satisfies the m/z error criterion abs(theor-mea)/theor < 10*10^(-6) (10 ppm) relative to the matched theoretical adduct m/z.
- The Annotator output file exists, is non-empty, and its first column contains the measured m/z values from MSIdata.
- Each annotated row pairs a metabolite name with an adduct form whose theoretical m/z lies within the 10 ppm tolerance of the measured m/z.
- The Database organ and detection mode (e.g. liver, positive) were confirmed consistent with MSIdata before running the script.
- Changing line 25 (e.g. from 10*10^(-6) to 5*10^(-6)) produces a correspondingly narrower set of annotations, confirming the threshold is being applied.

## Limitations

- The default 10 ppm m/z error range must be manually edited in line 25 of MSIannotator.m if a different tolerance is required; there is no command-line parameter.
- Annotation quality depends on the supplied database — the established database covers 1298 metabolites in positive mode and 515 in negative mode, so metabolites outside it cannot be annotated.
- Input sheets must be imported into MATLAB as string arrays renamed exactly MSIdata and Database; inconsistent naming or format will break the script.
- No changelog was found for the source repository, so version differences in behavior cannot be tracked.

## Evidence

- [abstract] An automated annotation tool, MSIannotator, was then created to conduct metabolite annotation in the MSI dataset with high efficiency and confidence.: "An automated annotation tool, MSIannotator, was then created to conduct metabolite annotation in the MSI dataset with high efficiency and confidence."
- [readme] if abs(theor-mea)/theor<10·10^(-6)%m/z error set as 10 ppm: "the default value of m/z error range is 10 ppm, and if necessary users can reset it by modifying the line 25 code "if abs(theor-mea)/theor<10·10^(-6)%m/z error set as 10 ppm""
- [intro] Import two sheets (one sheet is the preprocessed MSI dataset and the other is our provided database) into MATLAB in the format of a string array, and rename them as MSIdata and Database: "Import two sheets (one sheet is the preprocessed MSI dataset and the other is our provided database) into MATLAB in the format of a string array, and rename them as MSIdata and Database"
- [readme] the selected organ and the detection mode (positive or negative mode) should be consistent with that of MSIdata to be annotated: "To check whether the input Database is correct, the selected organ and the detection mode (positive or negative mode) should be consistent with that of MSIdata to be annotated."
- [intro] Generate a new file (Annotator), consisting of annotated metabolite names and their adduct forms.: "Generate a new file (Annotator), consisting of annotated metabolite names and their adduct forms. Users can copy the metabolite annotation results or save them."
