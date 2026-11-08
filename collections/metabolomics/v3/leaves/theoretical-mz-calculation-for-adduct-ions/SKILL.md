---
name: theoretical-mz-calculation-for-adduct-ions
description: Use when an established list of identified metabolites (e.g. organ-specific
  sheets from 'Organ specific metabolite database.xlsx') must be adduct-ion-expanded
  with theoretical m/z values to build or extend an annotation database for ambient
  mass spectrometry imaging (AFADESI-MSI) data.
when_to_use_negative:
- You already have a preprocessed MSI feature table and only need metabolite annotation
  — use MSIannotator.m directly with the provided Database.xlsx rather than rebuilding
  the database.
- No established metabolite identities exist yet (e.g. raw MSI data in discovery mode),
  since this skill expands known metabolite lists rather than identifying new features.
- The metabolite list, organ, or detection mode of the database does not match the
  MSI data to be annotated.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSIannotator.m
  role: MATLAB script that consumes the theoretical m/z columns of the database to
    annotate measured m/z features at a default 10 ppm error range (modifiable at
    line 25)
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Environment for importing the organ-specific metabolite sheets as string arrays,
    enumerating adduct forms, and computing theoretical m/z values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:dbbe323c91d9f460ad76fb9ecd1ef9c195fadd9aa018fa05db459196f81c045f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# theoretical-mz-calculation-for-adduct-ions

## Summary

Enumerates the adduct forms metabolites adopt in AFADESI-MSI (in contrast to their LC-MS adduct forms) and computes the theoretical m/z for every metabolite–adduct combination, expanding the organ-specific metabolite lists (1298 positive-mode and 515 negative-mode metabolites) into a metabolite database for MSI consisting of 27,407 adduct ions.

## When to use

Use when an established list of identified metabolites (e.g. organ-specific sheets from 'Organ specific metabolite database.xlsx') must be adduct-ion-expanded with theoretical m/z values to build or extend an annotation database for ambient mass spectrometry imaging (AFADESI-MSI) data.

## When NOT to use

- You already have a preprocessed MSI feature table and only need metabolite annotation — use MSIannotator.m directly with the provided Database.xlsx rather than rebuilding the database.
- No established metabolite identities exist yet (e.g. raw MSI data in discovery mode), since this skill expands known metabolite lists rather than identifying new features.
- The metabolite list, organ, or detection mode of the database does not match the MSI data to be annotated.

## Inputs

- 'Organ specific metabolite database.xlsx' (Sheet1: 1298 positive-mode metabolites; Sheet2: 515 negative-mode metabolites; Sheets 3–11: brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum organ-specific metabolite lists)
- Adduct-form-difference evaluation between LC-MS and AFADESI-MSI adduct forms
- Reference format from 'Database.xlsx' (metabolite names in first column, theoretical m/z in adduct-form columns)

## Outputs

- Adduct-ion-expanded metabolite database (e.g. adduct_ion_database.csv) with columns for metabolite name, adduct form, theoretical m/z, organ, and ionization mode, totaling 27,407 adduct ions
- Per-organ and per-mode theoretical m/z columns compatible with MSIannotator.m Database input

## How to apply

Load the released organ-specific metabolite lists from 'Organ specific metabolite database.xlsx', which contains 1298 metabolites in positive mode (Sheet1) and 515 metabolites in negative mode (Sheet2), partitioned into brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum sheets. For each organ/mode combination, enumerate the adduct forms observed for AFADESI-MSI, following the evaluation of the adduct form difference between LC-MS and AFADESI-MSI. Compute the theoretical m/z for every metabolite–adduct combination using standard adduct mass calculations, paralleling the theoretical m/z columns in Database.xlsx. Aggregate the enumerated adduct ions per organ sheet and mode, then sum across all sheets; the expanded database must total 27,407 adduct ions. These theoretical m/z columns are what downstream annotation (MSIannotator.m) matches measured m/z against within a default 10 ppm error range.

## Related tools

- **MSIannotator.m** (MATLAB script that consumes the theoretical m/z columns of the database to annotate measured m/z features at a default 10 ppm error range (modifiable at line 25)) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Environment for importing the organ-specific metabolite sheets as string arrays, enumerating adduct forms, and computing theoretical m/z values)

## Examples

```
if abs(theor-mea)/theor < 10*10^(-6) % m/z error set as 10 ppm
```

## Evaluation signals

- Total row count of the expanded adduct-ion database must equal the reported 27,407 adduct ions.
- Every metabolite in 'Organ specific metabolite database.xlsx' (1298 positive-mode, 515 negative-mode) appears in at least one adduct-ion row for its organ and mode.
- Spot-check computed theoretical m/z values against the theoretical m/z columns in Database.xlsx (e.g. liver positive mode).
- Each organ sheet and ionization mode is represented in the output; per-organ adduct-ion counts sum to the overall total.
- Downstream MSIannotator.m run with the generated database annotates the example liver positive-mode MSIdata within the 10 ppm tolerance without empty results.

## Limitations

- The database is organ- and ionization-mode-specific: the selected organ and detection mode of the database must be consistent with the MSI data being annotated.
- The adduct-ion count depends on the adduct-form-difference evaluation between LC-MS and AFADESI-MSI; adducts not observed in that evaluation are not enumerated.
- Downstream annotation uses a default m/z error range of 10 ppm; users must modify line 25 of MSIannotator.m if a different tolerance is needed.
- No changelog is available for the source repository, so database revisions are hard to track.

## Evidence
