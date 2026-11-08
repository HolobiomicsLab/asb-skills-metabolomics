---
name: msdial-feature-table-parsing
description: Use when an MS-DIAL output peak list from the CSH-Q Exactive HF lipidomics
  assay contains positive-ionization precursor m/z values that lack MS/MS-based annotation
  and need lipid hits assigned by accurate mass alone.
when_to_use_negative:
- The input is raw LC-MS/MS vendor data or mzML rather than an exported MS-DIAL peak/feature
  list — feature detection must happen first.
- The goal is MSI level 2 annotation by precursor accurate mass plus MS/MS matching
  (or MSI level 1 with authentic standards), not precursor-mass-only (level 3) lookup.
- The dataset comes from the HILIC-Q Exactive HF metabolomics assay rather than the
  CSH-Q Exactive HF lipidomics assay with positive-ionization precursor masses.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL
  role: Produces the lipidomics peak/feature list whose positive-ionization precursor
    m/z values are parsed and passed to the LipidBlast lookup
- name: LipidBlast v49
  role: In-silico lipid MS/MS library queried via its m/z lookup macro at a 5 mDa
    precursor mass tolerance
- name: CSH-Q Exactive HF mass spectrometry
  role: Instrument/assay that generated the lipidomics precursor masses being parsed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:ee45914a44aec367a1f441544d841872fb89ec91664ded7f5ec844c148771b9a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSH-Q Exactive HF mass spectrometry
  - LipidBlast v49
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# msdial-feature-table-parsing

## Summary

Parses MS-DIAL lipidomics peak/feature lists (precursor m/z values from the CSH-Q Exactive HF assay in positive ionization mode) and queries them against the LipidBlast v49 in-silico lipid MS/MS library with a 5 mDa precursor mass tolerance to recover additional MSI level 3 lipid annotations.

## When to use

Use when an MS-DIAL output peak list from the CSH-Q Exactive HF lipidomics assay contains positive-ionization precursor m/z values that lack MS/MS-based annotation and need lipid hits assigned by accurate mass alone.

## When NOT to use

- The input is raw LC-MS/MS vendor data or mzML rather than an exported MS-DIAL peak/feature list — feature detection must happen first.
- The goal is MSI level 2 annotation by precursor accurate mass plus MS/MS matching (or MSI level 1 with authentic standards), not precursor-mass-only (level 3) lookup.
- The dataset comes from the HILIC-Q Exactive HF metabolomics assay rather than the CSH-Q Exactive HF lipidomics assay with positive-ionization precursor masses.

## Inputs

- MS-DIAL lipidomics output peak/feature list from the CSH-Q Exactive HF assay (with precursor m/z and ionization-mode columns)
- LipidBlast v49 in-silico lipid MS/MS library with its m/z lookup macro

## Outputs

- lipid_blast_annotations.csv: a single annotation table with compound name, formula, precursor m/z, and mass error for each accepted lipid hit (96 annotations expected for the urinary lipidomics dataset)

## How to apply

Read the MS-DIAL lipidomics output (peak list) for the CSH-Q Exactive HF assay and retain precursor m/z values recorded in positive ionization mode. Convert each precursor mass and apply the LipidBlast v49 m/z lookup macro to search the in-silico lipid MS/MS library. Filter matches using the 5 mDa precursor mass tolerance specified for the lipid library search, then compile all accepted lipid hits into a single annotation table holding compound name, formula, precursor m/z, and mass error. The rationale is that precursor accurate mass lookup alone (MSI level 3) recovers lipid annotations for signals whose MS/MS spectra did not yield level 1 or level 2 assignments, complementing the SIRIUS/CSI:FingerID and NIST hybrid search workflow described in the paper.

## Related tools

- **MS-DIAL** (Produces the lipidomics peak/feature list whose positive-ionization precursor m/z values are parsed and passed to the LipidBlast lookup)
- **LipidBlast v49** (In-silico lipid MS/MS library queried via its m/z lookup macro at a 5 mDa precursor mass tolerance)
- **CSH-Q Exactive HF mass spectrometry** (Instrument/assay that generated the lipidomics precursor masses being parsed)

## Examples

```
lipid_hits_df[['compound_name','formula','precursor_mz','mass_error']].to_csv('lipid_blast_annotations.csv')
```

## Evaluation signals

- The count of accepted matches equals 96 — the number of additional lipid annotations reported for the CSH-Q Exactive urinary lipidomics dataset.
- Every row in lipid_blast_annotations.csv carries compound name, formula, precursor m/z, and mass error.
- All retained precursor m/z values are traceable to the positive-ionization entries of the MS-DIAL peak list.
- Each accepted hit's mass error is within the 5 mDa precursor mass tolerance specified for the lipid library search.
- No MS/MS-matched (MSI level 1/2) assignments are re-annotated — the table should contain only additional (previously unannotated) lipids.

## Limitations

- Matches are MSI level 3 annotations based on precursor accurate mass alone, so isobaric lipids within the 5 mDa window cannot be distinguished without MS/MS evidence.
- The paper notes the field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments, so these level 3 assignments should be treated as tentative.
- The article notes that the majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals, so mass-only lookup should be positioned as a complementary, not primary, annotation strategy.

## Evidence

- [other] Using the LipidBlast v49 m/z lookup macro with a 5 mDa mass tolerance on positive-ionization MS-DIAL precursor masses yielded 96 additional lipid annotations for the lipidomics data set.: [evidence span withheld — anchor status: not_found]
