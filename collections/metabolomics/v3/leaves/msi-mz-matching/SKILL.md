---
name: msi-mz-matching
description: Use when a preprocessed MSI feature list (measured m/z in column 1) must
  be annotated against an organ-specific metabolite database (metabolite name + theoretical
  m/z in specified adduct columns) with a controlled ppm tolerance.
when_to_use_negative:
- Input is raw, unprocessed MSI data rather than a preprocessed feature list whose
  first column already contains measured m/z — the script expects MSIdata already
  preprocessed.
- The organ sheet or detection mode of the supplied Database does not match the MSIdata
  to be annotated (e.g. a liver positive-mode database against a negative-mode dataset);
  the README explicitly requires checking this consistency before running.
- No user-supplied or built-in metabolite database with theoretical m/z in the required
  adduct forms exists for the target organ and ionization mode.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MSIannotator.m
  role: MATLAB script that performs the per-feature ppm matching of MSIdata m/z against
    theoretical adduct m/z in Database and writes the Annotator output
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which MSIdata and Database sheets are imported as string
    arrays and MSIannotator.m is executed
- name: MSIannotator
  role: The annotation tool (published as MSIannotator) created to conduct metabolite
    annotation in the MSI dataset with high efficiency and confidence
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:0a0f0e0fdc6199137e08052a0069d551dd0e1e3ed1cd9313c2d3887df9be4e14
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# msi-mz-matching

## Summary

Matches measured m/z features in a preprocessed mass spectrometry imaging (MSI) dataset against an organ-specific metabolite database using a ppm-based mass-error window, assigning each feature an annotated metabolite name and adduct form. Implemented as the MSIannotator.m MATLAB script, it underpins the AFADESI-MSI annotation workflow of Zhu et al. (Anal. Chem. 2022).

## When to use

Use when a preprocessed MSI feature list (measured m/z in column 1) must be annotated against an organ-specific metabolite database (metabolite name + theoretical m/z in specified adduct columns) with a controlled ppm tolerance.

## When NOT to use

- Input is raw, unprocessed MSI data rather than a preprocessed feature list whose first column already contains measured m/z — the script expects MSIdata already preprocessed.
- The organ sheet or detection mode of the supplied Database does not match the MSIdata to be annotated (e.g. a liver positive-mode database against a negative-mode dataset); the README explicitly requires checking this consistency before running.
- No user-supplied or built-in metabolite database with theoretical m/z in the required adduct forms exists for the target organ and ionization mode.

## Inputs

- MSIdata: preprocessed MSI dataset as a MATLAB string array, measured m/z in the first column (example: MSIdata.xlsx, liver tissue in positive mode)
- Database: metabolite database as a MATLAB string array, metabolite names in column 1 and theoretical m/z in specified adduct form columns (example: Database.xlsx, liver metabolite database in positive mode)
- Organ specific metabolite database.xlsx: established database of 1298 metabolites in positive mode (Sheet1) and 515 in negative mode (Sheet2), divided into brain, liver, kidney, heart, spleen, lung, muscle, pancreas and serum sheets (Sheets 3–11)
- m/z error tolerance setting in line 25 of MSIannotator.m (default 10*10^(-6) = 10 ppm; user-editable, e.g. 5*10^(-6) = 5 ppm)

## Outputs

- Annotator: annotated metabolite list file, with measured m/z in the first column and intermediate and ultimate annotated metabolite names plus their adduct forms in the following columns

## How to apply

Load the preprocessed MSI dataset into MATLAB as a string array named MSIdata and the database as a string array named Database; MSIdata must have measured m/z in its first column, and Database must list metabolite names in column 1 with theoretical m/z values in adduct-specific columns. First verify consistency: the selected organ sheet (e.g. liver, Sheet4 of Organ specific metabolite database.xlsx) and the detection mode (positive or negative) must match the MSIdata being annotated, since organ and mode determine which 1298 positive-mode or 515 negative-mode metabolites apply. Run MSIannotator.m, which for each measured m/z computes the relative mass error abs(theor-mea)/theor against every theoretical adduct m/z and accepts matches below the threshold; the default is 10*10^(-6) (10 ppm) set at line 25, and users can tighten it, e.g. to 5 ppm, by editing that line to abs(theor-mea)/theor<5·10^(-6). The rationale for a ppm (rather than absolute Da) window is that mass error scales with m/z, so a relative tolerance keeps matching rigorous across the full m/z range. Collect, for each accepted match, the intermediate and ultimate annotated metabolite name together with its adduct form, and write the results to the Annotator output file, ordered with measured m/z in column 1 and metabolite/adduct annotations in the following columns.

## Related tools

- **MSIannotator.m** (MATLAB script that performs the per-feature ppm matching of MSIdata m/z against theoretical adduct m/z in Database and writes the Annotator output) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which MSIdata and Database sheets are imported as string arrays and MSIannotator.m is executed)
- **MSIannotator** (The annotation tool (published as MSIannotator) created to conduct metabolite annotation in the MSI dataset with high efficiency and confidence) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
matlab -batch "MSIdata=readtable('MSIdata.xlsx'); Database=readtable('Database.xlsx'); run('MSIannotator.m')"
```

## Evaluation signals

- Every annotation reported in Annotator satisfies the configured mass-error criterion: abs(theor-mea)/theor < 10*10^(-6) (or the user-set value such as 5*10^(-6)) relative to the theoretical adduct m/z in Database.
- Each measured m/z appearing in Annotator column 1 corresponds to a feature in MSIdata column 1 — no invented or dropped m/z values.
- Annotator contains both intermediate and ultimate annotated metabolite names plus adduct forms, matching the declared output schema.
- The organ and detection mode of the Database used (e.g. liver, positive mode) match the MSIdata, as required before running the script.
- Editing line 25 of MSIannotator.m demonstrably changes the number of accepted matches, confirming the tolerance parameter took effect.

## Limitations

- The default 10 ppm m/z error range may admit false annotations; users must consciously reset it (e.g. to 5 ppm) by modifying line 25 for stricter matching.
- Matching is one-dimensional (m/z only, within a ppm window); no additional scoring, isotope, or MS/MS evidence is applied in this script.
- Only adduct forms pre-listed as theoretical m/z columns in Database can be matched; adducts absent from the database columns are never annotated.
- Database coverage is bounded by the established organ-specific metabolite database (1298 metabolites positive mode / 515 negative mode across 9 organ/serum sheets); metabolites outside it cannot be annotated.
- The source repository has no changelog, so tracking script revisions relies on manual inspection.

## Evidence
