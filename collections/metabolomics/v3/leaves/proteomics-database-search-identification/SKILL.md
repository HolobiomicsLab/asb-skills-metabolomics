---
name: proteomics-database-search-identification
description: Use when you have vendor .raw proteomics spectra (e.g. PXD021518, PXD023047)
  and need FDR-controlled peptide/spectrum identifications as ground truth for evaluating
  MS/MS clustering.
when_to_use_negative:
- Input is metabolomics data where FDR-controlled identifications are not currently
  practical — use the MS-RT retention-time method instead
- You already have FDR-filtered PSM identifications and only need to benchmark clustering
  against them
- Spectra are non-proteomics (metabolomics MS/MS) where a target-decoy peptide database
  search is not applicable
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MS-GF+
  role: Performs the target-decoy database search against the Arabidopsis thaliana
    proteome to produce PSM identifications with FDR estimates
- name: msconvert
  role: Converts vendor-specific Thermo .raw files to mzML format required as MS-GF+
    input
- name: ProteoWizard
  role: Toolkit that provides the msconvert command used for raw-to-mzML conversion
- name: Python
  role: Scripting environment for filtering PSMs to 1% FDR and writing the identified-PSM
    table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:c19293d3a773713e182aa8caba890d7e7952d3c267868d45f412c2bbaf4bf3df
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-GF+
  - ProteoWizard
  - Python
  - msconvert
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# proteomics-database-search-identification

## Summary

Runs MS-GF+ database search on mzML-converted Thermo .raw proteomics data and filters PSMs to 1% FDR via target-decoy, generating the identified-PSM ground-truth table that the MS-RT paper uses to evaluate MS/MS clustering.

## When to use

Use when you have vendor .raw proteomics spectra (e.g. PXD021518, PXD023047) and need FDR-controlled peptide/spectrum identifications as ground truth for evaluating MS/MS clustering.

## When NOT to use

- Input is metabolomics data where FDR-controlled identifications are not currently practical — use the MS-RT retention-time method instead
- You already have FDR-filtered PSM identifications and only need to benchmark clustering against them
- Spectra are non-proteomics (metabolomics MS/MS) where a target-decoy peptide database search is not applicable

## Inputs

- Vendor-specific Thermo .raw mass spectrometry files from ProteomeXchange datasets PXD021518 and PXD023047
- Arabidopsis thaliana UniProt reference proteome (target-decoy database)
- mzML-converted spectra files (required by MS-GF+)

## Outputs

- Filtered identified-PSM table with columns: filename, scan, precursor mass, retention time, peptide/protein identification
- 1% FDR-filtered PSM set stored in data/DB-Search directory
- Ground-truth purity labels for clustering_eval_compare_DB evaluation

## How to apply

First convert vendor-specific Thermo .raw files to mzML using msconvert from the ProteoWizard toolkit or ThermoRawFileParser. Run MS-GF+ (version 2023.01.12) against the Arabidopsis thaliana UniProt reference proteome in target-decoy mode with trypsin specificity, NumMods 2, and a 10 ppm precursor tolerance for PXD021518 (20 ppm for PXD023047). Filter the PSM output to 1% false discovery rate using the target-decoy q-value assignment from the search output. Write the filtered identified-PSM table (filename, scan, precursor mass, retention time, peptide/protein identification) into the data/DB-Search directory for use by the clustering_eval_compare_DB comparison step. The resulting PSM-level identifications serve as an external purity reference against which clustering tools like MS-Cluster, Falcon, and MaRaCluster are scored.

## Related tools

- **MS-GF+** (Performs the target-decoy database search against the Arabidopsis thaliana proteome to produce PSM identifications with FDR estimates)
- **msconvert** (Converts vendor-specific Thermo .raw files to mzML format required as MS-GF+ input)
- **ProteoWizard** (Toolkit that provides the msconvert command used for raw-to-mzML conversion)
- **Python** (Scripting environment for filtering PSMs to 1% FDR and writing the identified-PSM table)

## Examples

```
msconvert PXD021518.raw --mzML && ms-GF+ -i PXD021518.mzML -d arabidopsis.fasta -o psms.tsv
```

## Evaluation signals

- Confirm the output identified-PSM table exists and contains all required per-spectrum fields (filename, scan, precursor mass, retention time, peptide/protein identification)
- All retained PSMs pass the 1% FDR threshold implied by the target-decoy q-value filter
- MS-GF+ version matches 2023.01.12 and precursor tolerance matches the dataset-specific value (10 ppm for PXD021518, 20 ppm for PXD023047)
- The mzML conversion completed for every .raw file before search (no missing vendor formats)
- The filtered table lands in data/DB-Search and is consumable by the clustering_eval_compare_DB step without column-name mismatches

## Limitations

- Requires a well-annotated reference proteome; proteomics-only approach since metabolomics lacks FDR-controlled identifications for external ground truth
- FDR filtering depends on target-decoy q-value assignment from the search output — a malformed or missing q-value column makes the 1% cutoff unreliable
- Dataset-specific precursor tolerances differ (10 ppm vs 20 ppm), so parameters are not portable across PXD datasets without adjustment

## Evidence

- [methods] Proteomics DB-search FDR filtering: [evidence span withheld — anchor status: not_found]
