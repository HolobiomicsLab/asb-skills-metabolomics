---
name: reference-deposit-comparison
description: Use when a locally regenerated preprocessed MS-file (e.g., a '_SR' saturation-repair
  output in Agilent .d profile mode) must be validated against the authors' deposited
  preprocessed file in a public deposit such as MassIVE MSV000087295.
when_to_use_negative:
- No deposited reference preprocessed file exists for the dataset — without an accession
  like MSV000087295 there is no comparison target for output_matches_reference.
- The goal is new discovery (e.g., counting lipid annotations) rather than verifying
  that a reproduced preprocessing run matches published output.
- Data are already reduced to peak lists or feature tables, so instrument-format/profile-mode
  structural equivalence with the deposit cannot be assessed.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Source of the saturation repair module whose regenerated '_SR' output (same
    instrument format, profile mode, abundance threshold at 40% of the abundance limit)
    is compared against the deposited preprocessed reference.
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: R
  role: Computes the isotopic validation score as the Pearson correlation of the repaired
    isotopic distribution against ChemCalc theoretical distributions.
- name: ChemCalc
  role: Computes the theoretical isotopic distributions used as the target for the
    isotopic Pearson-correlation score.
- name: MassIVE
  role: Public data repository hosting the deposited original and preprocessed human
    plasma lipid-extract LC-DT IM-MS triplicates (MSV000087295) used as the comparison
    reference.
  repo: ftp://massive.ucsd.edu/MSV000087295
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:75b168c8437635f9fa81dc1ec4c626e7b5511e1c1bba05ffc6c40048c2615d38
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemCalc
  - MassIVE
  - PNNL PreProcessor
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# reference-deposit-comparison

## Summary

Validates a reproduced preprocessing output (e.g., PNNL PreProcessor saturation repair on Agilent .d LC-DT IM-MS lipid files) via direct checks — file existence, instrument format, structural match — against the authors' deposited preprocessed file in the MassIVE deposit MSV000087295, optionally augmented by an R/ChemCalc isotopic Pearson-correlation score.

## When to use

Use when a locally regenerated preprocessed MS-file (e.g., a '_SR' saturation-repair output in Agilent .d profile mode) must be validated against the authors' deposited preprocessed file in a public deposit such as MassIVE MSV000087295.

## When NOT to use

- No deposited reference preprocessed file exists for the dataset — without an accession like MSV000087295 there is no comparison target for output_matches_reference.
- The goal is new discovery (e.g., counting lipid annotations) rather than verifying that a reproduced preprocessing run matches published output.
- Data are already reduced to peak lists or feature tables, so instrument-format/profile-mode structural equivalence with the deposit cannot be assessed.

## Inputs

- Original raw human plasma lipid-extract LC-DT IM-MS triplicate MS-files in Agilent .d format (MassIVE MSV000087295)
- Deposited preprocessed MS-file from the same MassIVE deposit (MSV000087295), serving as the structural comparison reference
- Filtered MS-file that is input to the reproduced PreProcessor saturation repair module
- Theoretical isotopic distributions computed with ChemCalc (for the optional isotopic score)

## Outputs

- Regenerated preprocessed MS-file in the original instrument format, profile mode, with the '_SR' file-name suffix
- Direct-check verdicts: file_exists, file_format_is, and output_matches_reference (regenerated vs. deposited file)
- Isotopic Pearson-correlation scores computed in R against ChemCalc theoretical distributions

## How to apply

Retrieve both the original raw and the deposited preprocessed human plasma lipid-extract LC-DT IM-MS triplicate files (Agilent .d format) from ftp://massive.ucsd.edu/MSV000087295, then rerun the preprocessing step under study — here the PNNL PreProcessor saturation repair module on the filtered MS-file with the abundance threshold set to 40% of the abundance limit, keeping output in the same instrument format in profile mode and appending the '_SR' suffix. Run three direct checks on the regenerated file: file_exists (it is present at the expected path), file_format_is (it retains the expected instrument format and profile mode), and output_matches_reference (its structure matches the deposited preprocessed MS-file within format/tolerance). Repeat the comparison for all three replicates, since the plasma lipid extract was analyzed in triplicate. For quantitative confirmation, compute isotopic scores in R as the Pearson correlation of the repaired isotopic distribution against theoretical distributions computed with ChemCalc, targeting the paper's reported outcome of a tripled monoisotopic peak height and mass error reduced from 10.6 to 1.9 pm; note this correlation score is judged by expert review rather than an automated check.

## Related tools

- **PNNL PreProcessor** (Source of the saturation repair module whose regenerated '_SR' output (same instrument format, profile mode, abundance threshold at 40% of the abundance limit) is compared against the deposited preprocessed reference.) — https://omics.pnl.gov/software/pnnl-preprocessor
- **R** (Computes the isotopic validation score as the Pearson correlation of the repaired isotopic distribution against ChemCalc theoretical distributions.)
- **ChemCalc** (Computes the theoretical isotopic distributions used as the target for the isotopic Pearson-correlation score.)
- **MassIVE** (Public data repository hosting the deposited original and preprocessed human plasma lipid-extract LC-DT IM-MS triplicates (MSV000087295) used as the comparison reference.) — ftp://massive.ucsd.edu/MSV000087295

## Examples

```
Rscript -e 'score <- cor(repaired_iso_dist, chemcalc_theo_dist, method="pearson")'
```

## Evaluation signals

- file_exists: the '_SR'-suffixed saturation-repaired output file is present at the expected path after the PreProcessor run.
- file_format_is: the repaired output is in the expected instrument format and profile mode, matching the deposit.
- output_matches_reference: the regenerated preprocessed MS-file structurally matches the deposited preprocessed file from MSV000087295 within format/tolerance.
- All three deposited replicates are compared, not just one, since the human plasma lipid extract was analyzed by LC-DT IM-MS in triplicates.
- Isotopic Pearson correlation (R vs. ChemCalc theoretical distributions) supports the reported outcomes: tripled monoisotopic peak height and mass error reduced from 10.6 to 1.9 pm.

## Limitations

- output_matches_reference is a structural format/tolerance comparison, not bit-for-bit; a different PreProcessor version or a threshold other than 40% of the abundance limit will alter the output.
- The isotopic Pearson-correlation score is judged by expert review and depends on externally computed ChemCalc theoretical distributions.
- The comparison is demonstrated only for Agilent .d LC-DT IM-MS data; other IM-MS instrumentation requires vendor support or converting to UIMF as an intermediate step.

## Evidence
