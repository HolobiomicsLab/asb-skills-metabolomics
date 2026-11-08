---
name: insilico-lipid-library-search
description: Use when a positive-ion LC-HRMS lipidomics peak list contains unannotated
  precursor m/z values and you need additional lipid annotations beyond MSI level
  1/2 spectral matches, as MSI level 3 accurate-mass-only assignments.
when_to_use_negative:
- When the peak already has an MS/MS spectral match to an authentic standard or library
  spectrum (MSI level 1 or 2) — use that higher-confidence assignment instead.
- When precursor mass accuracy is worse than the library search tolerance (e.g. 5
  mDa), in-silico accurate-mass-only lookup cannot produce defensible MSI level 3
  assignments.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CSH-Q Exactive HF mass spectrometry
  role: Instrument that generates the precursor accurate masses for the in-silico
    lipid library lookup
- name: SIRIUS
  role: Framework for the in-silico fragmentation-based structure annotation workflow
    of which lipid library search is one component; supports precursor mass and MS/MS
    matching for level 2/3 annotation
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:7ad71794507946e64f715777baec234b35b597e2f4b98e3042e276bd43be6d9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# In-silico lipid library search

## Summary

Annotate lipid precursor masses from LC-HRMS peak lists by searching an in-silico lipid MS/MS library (e.g. LipidBlast v49) within a fixed precursor mass tolerance, yielding MSI level 3 annotations that extend spectral-matched identifications.

## When to use

Use when a positive-ion LC-HRMS lipidomics peak list contains unannotated precursor m/z values and you need additional lipid annotations beyond MSI level 1/2 spectral matches, as MSI level 3 accurate-mass-only assignments.

## When NOT to use

- When the peak already has an MS/MS spectral match to an authentic standard or library spectrum (MSI level 1 or 2) — use that higher-confidence assignment instead.
- When precursor mass accuracy is worse than the library search tolerance (e.g. 5 mDa), in-silico accurate-mass-only lookup cannot produce defensible MSI level 3 assignments.

## Inputs

- MS-DIAL peak list from untargeted lipidomics (e.g. CSH-Q Exactive HF assay) with precursor m/z values in positive ionization mode
- In-silico lipid MS/MS library (LipidBlast v49) m/z lookup macro

## Outputs

- Annotation table of additional lipid hits with compound name, formula, precursor m/z, and mass error (lipid_blast_annotations.csv)
- MSI level 3 lipid annotations count (e.g. 96 additional annotations)

## How to apply

Read the MS-DIAL output peak list from the untargeted lipidomics assay (e.g. the CSH-Q Exactive HF run) and retain precursor m/z values measured in positive ionization mode for peaks not yet annotated at MSI level 1 or 2. Apply the in-silico lipid library lookup (the LipidBlast v49 m/z lookup macro) to each precursor mass, searching the in-silico lipid MS/MS library. Filter candidate matches with the precursor mass tolerance specified for the lipid library search — 5 mDa in this study — rather than a looser ppm window. Compile accepted hits into a single annotation table with compound name, formula, precursor m/z, and mass error, and treat these as MSI level 3 annotations because only precursor accurate mass was used (no MS/MS matching). Rationale: in-silico libraries cover lipid classes for which authentic spectra are unavailable, allowing the majority of unannotated signals to receive a chemical class-level assignment.

## Related tools

- **CSH-Q Exactive HF mass spectrometry** (Instrument that generates the precursor accurate masses for the in-silico lipid library lookup)
- **SIRIUS** (Framework for the in-silico fragmentation-based structure annotation workflow of which lipid library search is one component; supports precursor mass and MS/MS matching for level 2/3 annotation) — https://github.com/sirius-ms/sirius

## Examples

```
python lipid_lookup.py --input msdial_lipidomics.csv --library LipidBlast_v49 --tol 0.005 --mode positive --output lipid_blast_annotations.csv
```

## Evaluation signals

- Count of accepted lipid matches equals the expected number of additional annotations (96 in the reference dataset).
- Each hit's mass error is within the 5 mDa precursor mass tolerance.
- Output table contains required columns: compound name, formula, precursor m/z, and mass error.
- No accepted match should be a compound already annotated at MSI level 1 or 2 in the same dataset.

## Limitations

- Annotations are MSI level 3: based on precursor accurate mass only, so structural isomers within a lipid class are not distinguished.
- Coverage is limited to lipids represented in the in-silico library (LipidBlast v49); unrepresented classes will be missed.
- The field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments in metabolomics MS/MS.

## Evidence

- [results] in-silico library search with 5 mDa tolerance yields additional MSI level 3 annotations: [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS is a java-based framework for LC-MS/MS analysis: [evidence span withheld — anchor status: external_doc]
