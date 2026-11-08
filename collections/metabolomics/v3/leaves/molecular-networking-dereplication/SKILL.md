---
name: molecular-networking-dereplication
description: Use when LC-MS/MS data of a bioactive fractionated natural extract is
  available (feature quantification table plus per-fraction bioassay values) and the
  goal is to dereplicate molecules via molecular networking and highlight bioactive
  candidates prior to isolation.
when_to_use_negative:
- Input is already a GNPS molecular network with library-annotated nodes — the score
  mapping adds nothing without per-fraction bioassay values.
- Only a single crude extract (no fractionation) was acquired, so there are no per-fraction
  bioassay values to correlate against.
- Direct-infusion or non-tandem MS data where MS/MS spectra needed for dereplication
  are absent.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MZmine2
  role: LC-MS/MS peak detection, deconvolution, alignment and CSV export of the feature
    quantification table
  repo: http://mzmine.github.io/
- name: Optimus
  role: Alternative OpenMS/KNIME pipeline for feature detection and quantification
    with the 'MS/MS filter' activated
  repo: https://github.com/MolecularCartography/Optimus
- name: Jupyter Notebook
  role: Runs the released notebook that computes Pearson correlations between feature
    abundances and bioassay values (requires R via IRkernel)
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: GNPS web-platform
  role: Runs the feature-based molecular networking (FBMN) Data Analysis job that
    dereplicates molecules and generates the network
  repo: http://gnps.ucsd.edu
- name: R
  role: Backend for the Pearson correlation and p-value correction inside the Jupyter
    notebook (IRkernel)
  repo: https://github.com/IRkernel/IRkernel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:d7eae41f3bfffde6b7cd6d8c162e547a70cbd969c3d46e12e0edb08b6b90e724
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS web-platform
  - Jupyter Notebook
  - MZmine2
  - Optimus
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# molecular-networking-dereplication

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Dereplicate molecules from an extract by MS/MS molecular networking (GNPS FBMN) before compound isolation, then annotate network nodes with bioactivity scores computed from feature abundances across fractions and the bioassay values of those fractions.

## When to use

Use when LC-MS/MS data of a bioactive fractionated natural extract is available (feature quantification table plus per-fraction bioassay values) and the goal is to dereplicate molecules via molecular networking and highlight bioactive candidates prior to isolation.

## When NOT to use

- Input is already a GNPS molecular network with library-annotated nodes — the score mapping adds nothing without per-fraction bioassay values.
- Only a single crude extract (no fractionation) was acquired, so there are no per-fraction bioassay values to correlate against.
- Direct-infusion or non-tandem MS data where MS/MS spectra needed for dereplication are absent.

## Inputs

- LC-MS/MS raw data files of the fractions (mzML from MZmine2 or Optimus/OpenMS)
- feature quantification table (features_quantification_matrix.csv) with aligned row ID, m/z, retention time and peak area per fraction
- bioassay experiment values for each fraction analyzed by LC-MS/MS (appended to produce features_quantification_matrix_edited_bioactivity.csv)

## Outputs

- correlation output table (features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv) with Pearson r2 and corrected p-values per feature
- GNPS FBMN molecular network (graphml/csv) with bioactivity-score-annotated nodes
- Cytoscape visualization exposing candidate bioactive molecules for isolation

## How to apply

First process LC-MS/MS data of the fractions with MZmine2 (v2.31, with the 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number' row filters) or Optimus (with the 'MS/MS filter' activated) to export a feature quantification table (row ID, m/z, retention time, peak area). Append the bioassay result of each fraction to this table (features_quantification_matrix.csv → features_quantification_matrix_edited_bioactivity.csv). Run the released Jupyter notebook from DorresteinLaboratory/Bioactive_Molecular_Networks, which computes a Pearson correlation score and its corrected significance (r2 > 0.8, p-value < 0.05) for every detected feature against the bioassay result, yielding a bioactivity score prediction. Finally, run a GNPS feature-based molecular networking (FBMN) Data Analysis job and visualize the correlation-annotated network in Cytoscape, so that nodes corresponding to potentially bioactive molecules are exposed for targeted isolation.

## Related tools

- **MZmine2** (LC-MS/MS peak detection, deconvolution, alignment and CSV export of the feature quantification table) — http://mzmine.github.io/
- **Optimus** (Alternative OpenMS/KNIME pipeline for feature detection and quantification with the 'MS/MS filter' activated) — https://github.com/MolecularCartography/Optimus
- **Jupyter Notebook** (Runs the released notebook that computes Pearson correlations between feature abundances and bioassay values (requires R via IRkernel)) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **GNPS web-platform** (Runs the feature-based molecular networking (FBMN) Data Analysis job that dereplicates molecules and generates the network) — http://gnps.ucsd.edu
- **R** (Backend for the Pearson correlation and p-value correction inside the Jupyter notebook (IRkernel)) — https://github.com/IRkernel/IRkernel

## Examples

```
Download the Jupyter notebook from the GitHub: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/ and run: jupyter notebook 2_correlation_analysis.ipynb --input features_quantification_matrix_edited_bioactivity.csv --output features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv
```

## Evaluation signals

- features_quantification_matrix.csv must contain every row ID, m/z, retention time and peak area exported from MZmine2/Optimus, and its edited counterpart must carry one bioassay value per analyzed fraction.
- The notebook's output table must contain a Pearson r2 and a corrected p-value for every detected feature, with significant candidates satisfying r2 > 0.8 and p < 0.05.
- GNPS FBMN job must succeed and the resulting network must include nodes only for features that had MS/MS scans (GNPS) — the MZmine2 row filter guarantees this.
- Candidate molecules highlighted in the network can be cross-checked against the list of compounds recovered by classical bioassay-guided fractionation; newly flagged molecules constitute the expected differentiator.
- Cytoscape network must render bioactivity scores as node attributes, allowing visual identification of high-correlation clusters.

## Limitations

- MS/MS validation of putative annotations is not provided in Optimus (only mz-RT level-2 matching to a molecules-of-interest list).
- The workflow requires bioassay values for every fraction analyzed by LC-MS/MS; missing bioassay data breaks the correlation step.
- Significance depends on the corrected p-value cutoff (p < 0.05) and correlation cutoff (r2 > 0.8); borderline candidates may be missed.
- The Jupyter notebook requires an R installation (and IRkernel), and Anaconda is recommended as the environment manager.

## Evidence
