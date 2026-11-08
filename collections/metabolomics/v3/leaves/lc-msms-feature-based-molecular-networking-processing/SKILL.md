---
name: lc-msms-feature-based-molecular-networking-processing
description: Use when the analysis goal is to detect, align, quantify and dereplicate
  LC-MS/MS features from natural product extract fractions in order to generate an
  MS/MS-annotated feature table for downstream bioactivity-correlation mapping (e.g.,
  antiviral screening).
when_to_use_negative:
- Input is already a fully aligned, MS/MS-annotated feature table ready for correlation
  analysis
- Direct-infusion data or spectra without chromatographic retention time
- MS/MS spectra are not acquired for the features of interest (the MZmine2 row filter
  'Keep only features with MS/MS scan (GNPS)' would remove all rows)
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: MZmine2
  role: Detects, aligns and quantifies LC-MS/MS features and exports the aligned feature
    quantification table and MS/MS spectral summary .MGF file
  repo: http://mzmine.github.io/
- name: Optimus
  role: KNIME/OpenMS alternative to MZmine2 for LC-MS feature detection, quantification,
    filtering and export of MS/MS-filtered features
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Underlying LC-MS feature detection and quantification algorithms used by the
    Optimus workflow
- name: Jupyter notebook
  role: Runs the Bioactive_Molecular_Networks notebook with R kernel to compute Pearson
    correlation scores between feature abundance and bioassay results
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: R
  role: Statistical language required by the Jupyter notebook (installed via IRkernel)
    to compute correlation significance
  repo: https://github.com/IRkernel/IRkernel
- name: GNPS web-platform
  role: Runs the feature-based molecular networking (FBMN) Data Analysis job using
    the .MGF and feature table; generates the molecular network for dereplication
  repo: http://gnps.ucsd.edu
- name: Cytoscape
  role: Imports the GNPS molecular network and the correlation output table, visualises
    nodes passing r² > 0.8 / p < 0.02 thresholds, and displays pie charts of ion distribution
    across samples
- name: Anaconda
  role: Environment manager recommended for installing Python/R dependencies needed
    by the notebook
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:5a09f399f3bff23031e80f980073913a2e76e76353d3260c8fa135130d147dd6
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Agilent 6560 IM-Q-ToF MS
  - Anaconda
  - Bruker Q-ToF
  - Cytoscape
  - GNPS
  - GNPS web-platform
  - Jupyter notebook
  - MZmine2
  - OpenMS
  - Optimus
  - R
  - Skyline
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- lc-msms-feature-based-molecular-networking
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# lc-msms-feature-based-molecular-networking-processing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A LC-MS/MS feature-based molecular networking processing skill that converts raw chromatographic runs into an aligned, MS/MS-annotated feature quantification table (MZmine2) and/or processed features (Optimus) ready for downstream bioactivity correlation and GNPS FBMN, enabling dereplication and mapping of bioactive molecules in natural extract fractions.

## When to use

Use when the analysis goal is to detect, align, quantify and dereplicate LC-MS/MS features from natural product extract fractions in order to generate an MS/MS-annotated feature table for downstream bioactivity-correlation mapping (e.g., antiviral screening).

## When NOT to use

- Input is already a fully aligned, MS/MS-annotated feature table ready for correlation analysis
- Direct-infusion data or spectra without chromatographic retention time
- MS/MS spectra are not acquired for the features of interest (the MZmine2 row filter 'Keep only features with MS/MS scan (GNPS)' would remove all rows)

## Inputs

- LC-MS/MS raw runs of natural extract fractions (e.g., Euphorbia dendroides)
- Bioassay results per fraction (e.g., antiviral values)
- MZmine2 or Optimus configuration for feature detection/alignment

## Outputs

- features_quantification_matrix.csv (aligned feature list + intensity across fractions)
- features_quantification_matrix_edited_bioactivity.csv (feature table with bioassay values appended)
- MS/MS spectral summary .MGF file
- Exported CSV (row ID, m/z, RT, peak area)
- GNPS FBMN network + Cytoscape visualization with pie charts of ion distribution
- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv (correlation output table)

## How to apply

Process LC-MS/MS runs in MZmine2 (version 2.31) following the GNPS FBMN pipeline: apply the required 'Peak list row filter' options ('Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number') before export, then export via the 'Export to CSV' module (Export row ID, Export row m/z, Export row retention time, Peak area) plus the aligned features_quantification_matrix.csv and the MS/MS spectral summary .MGF file. If using Optimus (KNIME/OpenMS) instead, activate the 'MS/MS filter' option. Add the measured bioassay value for each fraction to the feature table, producing features_quantification_matrix_edited_bioactivity.csv, then run the Bioactive_Molecular_Networks Jupyter notebook (R kernel via IRkernel) to compute Pearson correlation scores and significance (r² > 0.8, p < 0.02) per feature against the bioassay. Submit the .MGF and feature table to the GNPS web-platform (http://gnps.ucsd.edu) as a feature-based molecular networking Data Analysis job and import the network into Cytoscape (min 3.4) for visualising nodes that pass the correlation threshold.

## Related tools

- **MZmine2** (Detects, aligns and quantifies LC-MS/MS features and exports the aligned feature quantification table and MS/MS spectral summary .MGF file) — http://mzmine.github.io/
- **Optimus** (KNIME/OpenMS alternative to MZmine2 for LC-MS feature detection, quantification, filtering and export of MS/MS-filtered features) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Underlying LC-MS feature detection and quantification algorithms used by the Optimus workflow)
- **Jupyter notebook** (Runs the Bioactive_Molecular_Networks notebook with R kernel to compute Pearson correlation scores between feature abundance and bioassay results) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **R** (Statistical language required by the Jupyter notebook (installed via IRkernel) to compute correlation significance) — https://github.com/IRkernel/IRkernel
- **GNPS web-platform** (Runs the feature-based molecular networking (FBMN) Data Analysis job using the .MGF and feature table; generates the molecular network for dereplication) — http://gnps.ucsd.edu
- **Cytoscape** (Imports the GNPS molecular network and the correlation output table, visualises nodes passing r² > 0.8 / p < 0.02 thresholds, and displays pie charts of ion distribution across samples)
- **Anaconda** (Environment manager recommended for installing Python/R dependencies needed by the notebook)

## Examples

```
MZmine2 (v2.31): run the GNPS FBMN pipeline with Peak list row filter options 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number', then 'Export to CSV' with row ID, m/z, retention time and Peak area; open Bioactive_molecular_networks_v1.0.ipynb with the R kernel (`IRkernel::installspec()`); submit the .MGF and features_quantification_matrix_edited_bioactivity.csv to http://gnps.ucsd.edu as an FBMN Data Analysis job; import the resulting network and correlation table into Cytoscape and select nodes with r² > 0.8 and p < 0.02.
```

## Evaluation signals

- The 'Peak list row filter' options 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number' must be activated in MZmine2 before export, otherwise the .MGF spectral summary will lack the required MS/MS scans
- The exported CSV must contain exactly the four columns 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area' as defined in the MZmine 'Export to CSV' module
- features_quantification_matrix_edited_bioactivity.csv must be produced by appending a bioassay value column to features_quantification_matrix.csv for each LC-MS/MS fraction
- Running the Jupyter notebook must yield features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv, containing Pearson correlation scores and corrected p-values for every detected feature
- In Cytoscape, selecting nodes with r² > 0.8 and p-value < 0.02 should expose the known antiviral candidate molecules reported for the Euphorbia dendroides extract, confirming that the bioactivity score mapping is reproducible

## Limitations

- MS/MS validation of putative annotations is not provided inside Optimus and must be done separately
- The Optimus workflow does not automatically clean up temporary files; iterative re-execution may require manual cleanup
- KNIME 2 GB RAM is the minimal requirement, which is generally insufficient for large datasets (hundreds of LC-MS runs); performance depends on instrument, mass resolution and LC run-time
- Only 64-bit operating systems are supported by Optimus
- The workflow requires installing R and the IRkernel inside the Jupyter environment; Anaconda is recommended as an environment manager
- The GNPS web-platform FBMN Data Analysis job does not use MS-Cluster for this workflow
- MZmine2 version must be 2.31 as per the published pipeline

## Evidence

- [readme] Optimus workflow for LC-MS-based untargeted metabolomics: [evidence span withheld — anchor status: external_doc]
- [readme] Exclusion of features for which MS/MS spectra were not acquired (optional step in Optimus): [evidence span withheld — anchor status: external_doc]
