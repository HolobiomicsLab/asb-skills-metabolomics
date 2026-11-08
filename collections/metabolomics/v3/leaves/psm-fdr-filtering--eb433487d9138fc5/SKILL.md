---
name: psm-fdr-filtering
description: Use when proteomics raw MS/MS data has been searched against a reference
  proteome with MS-GF+ and a target-decoy identified-PSM table filtered at 1% FDR
  is needed as an external evaluation reference for spectrum clustering.
when_to_use_negative:
- Metabolomics datasets where FDR-controlled molecule identifications are not practical
  and MS-RT (retention-time-based) evaluation must be used instead
- Datasets that have already been filtered to a target FDR and only need downstream
  clustering evaluation rather than ground-truth generation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS-GF+
  role: Performs the target-decoy database search whose PSM q-values enable 1% FDR
    filtering; version 2023.01.12 is specified in the source card workflow
- name: msconvert
  role: ProteoWizard tool that converts vendor-specific Thermo .raw files to mzML,
    the input format required before running the MS-GF+ search
  repo: http://proteowizard.sourceforge.net/download.html
- name: Python
  role: Used to write and validate the filtered identified-PSM table into data/DB-Search
    for downstream clustering comparison
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:21f17fafcfd861a9716802f1dcdf648f24c53393959e46b0523af5561ec25fd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-GF+
  - ProteoWizard
  - Python
  - msconvert
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# psm-fdr-filtering

## Summary

Filters peptide-spectrum matches (PSMs) from an MS-GF+ target-decoy database search to a 1% false discovery rate at the PSM level, yielding the identified-PSM table that serves as ground-truth purity labels for evaluating MS/MS clustering.

## When to use

Use when proteomics raw MS/MS data has been searched against a reference proteome with MS-GF+ and a target-decoy identified-PSM table filtered at 1% FDR is needed as an external evaluation reference for spectrum clustering.

## When NOT to use

- Metabolomics datasets where FDR-controlled molecule identifications are not practical and MS-RT (retention-time-based) evaluation must be used instead
- Datasets that have already been filtered to a target FDR and only need downstream clustering evaluation rather than ground-truth generation

## Inputs

- Raw Thermo .raw mass spectrometry files converted to mzML via msconvert or ThermoRawFileParser
- Arabidopsis thaliana UniProt reference proteome database in target-decoy form
- MS-GF+ search parameters: enzyme specificity (trypsin), maximum modifications (NumMods 2), precursor mass tolerance (10 ppm or 20 ppm)

## Outputs

- Filtered identified-PSM table with columns filename, scan, precursor mass, retention time, peptide/protein identification
- Target-decoy q-value annotation for every retained PSM confirming ≤1% FDR

## How to apply

Run MS-GF+ (version 2023.01.12) against a UniProt reference proteome in target-decoy mode with trypsin specificity, NumMods 2, and a precursor tolerance of 10 ppm (e.g. PXD021518) or 20 ppm (e.g. PXD023047). The search assigns a target-decoy q-value to each PSM, which encodes the estimated false discovery rate. Retain only PSMs whose q-value passes the 1% FDR threshold — this is the article's proteomics gold standard because, unlike metabolomics, 'FDR controlled identifications' are 'well-established' there. Write the filtered identified-PSM table (columns: filename, scan, precursor mass, retention time, peptide/protein identification) into the data/DB-Search directory of the Metabolomics_Clustering_Benchmark project so it can be consumed by the clustering_eval_compare_DB comparison step alongside MS-RT results.

## Related tools

- **MS-GF+** (Performs the target-decoy database search whose PSM q-values enable 1% FDR filtering; version 2023.01.12 is specified in the source card workflow)
- **msconvert** (ProteoWizard tool that converts vendor-specific Thermo .raw files to mzML, the input format required before running the MS-GF+ search) — http://proteowizard.sourceforge.net/download.html
- **Python** (Used to write and validate the filtered identified-PSM table into data/DB-Search for downstream clustering comparison) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark

## Examples

```
msconvert your_file.raw --mzML  # then: run MS-GF+ (v2023.01.12) with target-decoy, trypsin, NumMods 2, 10 ppm precursor tolerance
```

## Evaluation signals

- The output identified-PSM table exists and contains the required per-spectrum fields: filename, scan, precursor mass, retention time, and peptide/protein identification
- Every retained PSM carries a target-decoy q-value consistent with the 1% FDR cutoff
- The filtered table is located in data/DB-Search and is compatible with the column schema expected by clustering_eval_compare_DB
- The precursor tolerance used matches the dataset-specific value (10 ppm for PXD021518, 20 ppm for PXD023047), confirming correct search configuration

## Limitations

- This FDR-based ground truth only works for proteomics; the article notes that in mass spectrometry-based metabolomics 'FDR controlled identifications are not currently practical', so MS-RT must be substituted
- The source card specifies precursor tolerances per dataset (10 ppm vs 20 ppm); applying a single tolerance across datasets would misrepresent the search
- MS-GF+ version 2023.01.12 is the documented version — other versions may produce different q-value distributions and thus a different set of retained PSMs

## Evidence

- [methods] Target-decoy q-value filter at 1% FDR: [evidence span withheld — anchor status: not_found]
