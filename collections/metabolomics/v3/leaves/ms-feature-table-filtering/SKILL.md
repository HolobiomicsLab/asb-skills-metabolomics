---
name: ms-feature-table-filtering
description: Use when an aligned feature quantification matrix (features_quantification_matrix.csv
  from MZmine2 or Optimus) contains many features lacking MS/MS scans and must be
  reduced to MS/MS-backed rows before GNPS FBMN or bioactivity correlation.
when_to_use_negative:
- The input table is already the edited bioactivity-annotated file (features_quantification_matrix_edited_bioactivity.csv)
  with MS/MS filtering completed upstream.
- The dataset consists of direct-infusion data acquired without MS/MS scans, so the
  MS/MS row filter would empty the table.
- The analysis targets spatial mapping in Optimus rather than GNPS feature-based molecular
  networking, where MS/MS presence is not required.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine2
  role: Performs the 'Peak list row filter' step — 'Keep only features with MS/MS
    scan (GNPS)' and 'Reset the peak number' — then exports the filtered rows via
    'Export to CSV'.
  repo: http://mzmine.github.io/
- name: Optimus
  role: Alternative KNIME/OpenMS workflow whose 'MS/MS filter' option must be activated
    to exclude features lacking MS/MS spectra.
  repo: https://github.com/MolecularCartography/Optimus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:dc9ab393eaa77b7179f2a9c88b7e6a08e6d8a5186de895991076c963556d97f2
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MZmine2
  - Optimus
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ms-feature-table-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Filter the aligned LC-MS/MS feature quantification table produced by MZmine2 or Optimus so that only features with an acquired MS/MS scan survive for downstream bioactive molecular networking and Pearson-correlation scoring against bioassay values.

## When to use

Use when an aligned feature quantification matrix (features_quantification_matrix.csv from MZmine2 or Optimus) contains many features lacking MS/MS scans and must be reduced to MS/MS-backed rows before GNPS FBMN or bioactivity correlation.

## When NOT to use

- The input table is already the edited bioactivity-annotated file (features_quantification_matrix_edited_bioactivity.csv) with MS/MS filtering completed upstream.
- The dataset consists of direct-infusion data acquired without MS/MS scans, so the MS/MS row filter would empty the table.
- The analysis targets spatial mapping in Optimus rather than GNPS feature-based molecular networking, where MS/MS presence is not required.

## Inputs

- Aligned LC-MS/MS feature quantification matrix from MZmine2 or Optimus (features_quantification_matrix.csv with Export row ID, Export row m/z, Export row retention time, Peak area)

## Outputs

- Filtered feature table containing only features with MS/MS scans (input to the GNPS FBMN job and the Pearson-correlation Jupyter notebook)

## How to apply

In MZmine2 (version 2.31), open the 'Peak list row filter' module and enable the option 'Keep only features with MS/MS scan (GNPS)' so that only features carrying an associated MS/MS spectrum pass the filter; also apply 'Reset the peak number' to renumber the surviving rows consistently. In Optimus (KNIME/OpenMS), activate the 'MS/MS filter' option as an equivalent exclusion of features without MS/MS spectra. Next, export the filtered table via the 'Export to CSV' module in MZmine, including 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area', producing features_quantification_matrix.csv. The rationale is that the downstream Jupyter notebook computes Pearson correlations between feature intensities across fractions and bioassay results, and the GNPS feature-based molecular networking workflow requires MS/MS scans for spectral edges — retaining MS/MS-less features would dilute the network and inflate multiple-testing burden in the correlation step.

## Related tools

- **MZmine2** (Performs the 'Peak list row filter' step — 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number' — then exports the filtered rows via 'Export to CSV'.) — http://mzmine.github.io/
- **Optimus** (Alternative KNIME/OpenMS workflow whose 'MS/MS filter' option must be activated to exclude features lacking MS/MS spectra.) — https://github.com/MolecularCartography/Optimus

## Examples

```
# In MZmine2: Peak list row filter → 'Keep only features with MS/MS scan (GNPS)' + 'Reset the peak number' → 'Export to CSV' → features_quantification_matrix.csv
```

## Evaluation signals

- Row count of the exported CSV is strictly smaller than the unfiltered aligned feature list.
- Every retained row carries a non-null MS/MS scan identifier (the MZmine 'Keep only features with MS/MS scan (GNPS)' option was applied).
- Exported columns match the required schema: Export row ID, Export row m/z, Export row retention time, and Peak area.
- Peak numbers in the filtered table are contiguous after 'Reset the peak number' (no gaps from removed rows).
- The filtered table is accepted as input by the GNPS FBMN Data Analysis job without missing-scan errors.

## Limitations

- Features without MS/MS scans are discarded even if they show strong bioactivity correlation, so potentially bioactive molecules lacking fragmentation spectra are lost.
- MZmine2 and Optimus differ in the exact filter option names, so the steps are tool-specific and not directly interchangeable.
- The filter must be re-applied after any re-alignment of LC-MS/MS runs; a stale filtered table can mismatch the current feature list.
- Temporary files created by Optimus during execution are not deleted automatically and can occupy several times the initial dataset size.

## Evidence
