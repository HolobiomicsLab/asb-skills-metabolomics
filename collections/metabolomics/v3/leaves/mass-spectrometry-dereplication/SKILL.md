---
name: mass-spectrometry-dereplication
description: Use when you have a feature quantification table and an .MGF MS/MS summary
  from LC-MS/MS fractionation and must prioritize or annotate molecules before investing
  in purification.
when_to_use_negative:
- Input is already a curated feature table with library annotations — run the correlation/bioactivity
  mapping step instead
- The goal is full structure elucidation of an unknown, not putative dereplication
  — GNPS library matching alone is insufficient
- Data are direct infusion spectra without aligned LC features from a fractionation
  series — feature-based molecular networking inputs (row ID, m/z, RT, peak area)
  are not available
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MZmine2
  role: LC-MS/MS processing to produce the feature quantification table and .MGF MS/MS
    summary that are submitted to GNPS FBMN; applies the 'Keep only features with
    MS/MS scan (GNPS)' row filter and 'Export to CSV' module.
  repo: http://mzmine.github.io/
- name: GNPS web-platform
  role: Runs the feature-based molecular networking (FBMN) Data Analysis job, matching
    MS/MS spectra against libraries for dereplication prior to isolation.
  repo: http://gnps.ucsd.edu
- name: Optimus
  role: Alternative OpenMS-based LC-MS feature detection and quantification workflow
    (KNIME) that can replace MZmine2; requires the 'MS/MS filter' option to be activated
    so MS/MS spectra are exported for GNPS.
  repo: https://github.com/MolecularCartography/Optimus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:d0a31f8d3575f5d2d177f6f98e2d4b9c1ce19ba535ae6a42c6ae161b36186bb9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS web-platform
  - MZmine2
  - Optimus
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mass-spectrometry-dereplication

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Accelerates dereplication of molecules in a bioactive extract by submitting MS/MS spectra (MGF from MZmine2 or Optimus) to the GNPS feature-based molecular networking (FBMN) workflow so compounds are annotated in silico before isolation.

## When to use

Use when you have a feature quantification table and an .MGF MS/MS summary from LC-MS/MS fractionation and must prioritize or annotate molecules before investing in purification.

## When NOT to use

- Input is already a curated feature table with library annotations — run the correlation/bioactivity mapping step instead
- The goal is full structure elucidation of an unknown, not putative dereplication — GNPS library matching alone is insufficient
- Data are direct infusion spectra without aligned LC features from a fractionation series — feature-based molecular networking inputs (row ID, m/z, RT, peak area) are not available

## Inputs

- LC-MS/MS raw data of fractionated bioactive extract (mzML/RAW)
- Feature quantification table from MZmine2/Optimus (features_quantification_matrix.csv)
- .MGF file containing MS/MS spectral summaries (peak area, m/z, RT per feature)

## Outputs

- GNPS FBMN molecular network with putatively annotated compound nodes (library hits)
- Feature quantification table aligned across fractions (Export to CSV output)
- Cytoscape-importable GNPS network files for further bioactivity annotation

## How to apply

Process the LC-MS/MS data in MZmine2 (or Optimus with the MS/MS filter activated) to produce a feature quantification table and an .MGF file; in MZmine2 the 'Peak list row filter' must be set to 'Keep only features with MS/MS scan (GNPS)' with 'Reset the peak number', and features are exported via the 'Export to CSV' module (row ID, row m/z, row retention time, Peak area). Submit the .MGF to a GNPS Data Analysis job running the feature-based molecular networking (FBMN) workflow so the molecular network is built directly from the MGF rather than via MS-Cluster. GNPS matches spectra against public libraries to putatively annotate compounds across the network, letting dereplication precede downstream bioactivity mapping (Pearson correlation, r2 > 0.8, p-value < -0.02) and compound isolation.

## Related tools

- **MZmine2** (LC-MS/MS processing to produce the feature quantification table and .MGF MS/MS summary that are submitted to GNPS FBMN; applies the 'Keep only features with MS/MS scan (GNPS)' row filter and 'Export to CSV' module.) — http://mzmine.github.io/
- **GNPS web-platform** (Runs the feature-based molecular networking (FBMN) Data Analysis job, matching MS/MS spectra against libraries for dereplication prior to isolation.) — http://gnps.ucsd.edu
- **Optimus** (Alternative OpenMS-based LC-MS feature detection and quantification workflow (KNIME) that can replace MZmine2; requires the 'MS/MS filter' option to be activated so MS/MS spectra are exported for GNPS.) — https://github.com/MolecularCartography/Optimus

## Examples

```
Run a GNPS FBMN job: upload features_quantification_matrix_transposed.csv + .MGF to http://gnps.ucsd.edu, select 'FEATURE-BASED MOLECULAR NETWORKING', then in Cytoscape import bioactive_molecular_network session.
```

## Evaluation signals

- The .MGF file supplied to GNPS contains MS/MS spectra for all features retained after the 'Keep only features with MS/MS scan (GNPS)' filter — features lacking MS/MS scans should be absent
- The exported CSV from MZmine contains exactly the columns 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area'
- The GNPS FBMN job produces a molecular network whose nodes correspond to features in the feature quantification table (row IDs match), enabling later joining of Pearson correlation attributes
- Library-based putative annotations appear on network nodes, indicating dereplication occurred before isolation rather than after purification
- The workflow relies on open bioinformatic tools (MZmine2/Optimus, Jupyter notebook, GNPS) — no proprietary dereplication step should be present

## Limitations

- GNPS-based dereplication yields putative annotations only; MS/MS validation of putative level-2 Metabolomics Standards Initiative identifications is needed and is not provided by Optimus
- The 'MS/MS filter' option in Optimus must be activated or spectra will not be exported for downstream GNPS matching
- Dereplication is useful only if the extract has been fractionated and MS/MS acquired; the method was developed for dereplication of molecules prior to compound isolation, not for already-purified compounds

## Evidence

- [readme] Optimus optional MS/MS spectra acquisition filter: [evidence span withheld — anchor status: external_doc]
- [readme] GNPS library matching provides putative level-2 annotation: [evidence span withheld — anchor status: external_doc]
- [readme] MS/MS validation needed for putative annotation: [evidence span withheld — anchor status: external_doc]
