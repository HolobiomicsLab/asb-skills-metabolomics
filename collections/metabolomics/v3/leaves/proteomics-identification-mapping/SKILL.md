---
name: proteomics-identification-mapping
description: Use when clustering quality of MS/MS spectral clustering must be quantified
  against known peptide identifications, e.g., sweeping HyperSpec parameters like
  HD quantization level Q or dimension D and assembling a metric table per parameter
  value.
when_to_use_negative:
- The goal is peptide identification itself from unclustered spectra rather than evaluating
  clustering quality against existing MS-GF+ identifications
- The dataset lacks trustworthy spectrum-to-peptide labels (e.g., no MassIVE/PRIDE
  reanalysis identifications), so incorrect clustering ratio and completeness cannot
  be computed
- Comparing runtime/speedup performance of HyperSpec instead of clustering quality
  — runtime evaluation does not require identification mapping
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: HyperSpec
  role: Produces the per-Q clustering results (parquet cluster labels) that are mapped
    to MS-GF+ identifications for quality scoring; invoked with --hd_dim 2048 and
    varying --hd_Q
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Python
  role: Runs HyperSpec via python src/main.py and assembles the per-Q metric table
    from clustering output and identification labels
- name: ThermoRawFileParser
  role: Converts PXD000561 PRIDE raw files to MGF format as the input spectra for
    clustering and downstream quality evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:6ef41456cabe40a0adccaee44372599992c187a25dc8a2f87f783d4e2a48e005
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - CuPy
  - HyperSpec
  - Python
  - ThermoRawFileParser
  - cuML
  - fastcluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# proteomics-identification-mapping

## Summary

Maps clustered spectra back to peptide identifications (MS-GF+ results from MassIVE reanalysis RMSV000000091.3 for PXD000561) to compute clustering quality metrics — clustered spectra ratio, incorrect clustering ratio, and entropy-based completeness — used to evaluate HyperSpec hypervector generation parameters (e.g., a Q sweep of 4–64 at fixed D = 2048).

## When to use

Use when clustering quality of MS/MS spectral clustering must be quantified against known peptide identifications, e.g., sweeping HyperSpec parameters like HD quantization level Q or dimension D and assembling a metric table per parameter value.

## When NOT to use

- The goal is peptide identification itself from unclustered spectra rather than evaluating clustering quality against existing MS-GF+ identifications
- The dataset lacks trustworthy spectrum-to-peptide labels (e.g., no MassIVE/PRIDE reanalysis identifications), so incorrect clustering ratio and completeness cannot be computed
- Comparing runtime/speedup performance of HyperSpec instead of clustering quality — runtime evaluation does not require identification mapping

## Inputs

- MGF-format spectra files converted from PXD000561 PRIDE raw files (ThermoRawFileParser)
- MS-GF+ spectrum identifications from MassIVE reanalysis RMSV000000091.3 (charges 2 and 3) serving as peptide labels
- HyperSpec clustering results (parquet with cluster, is_representative, precursor_charge, identifier, scan columns) per Q setting
- Sweep values: hd_Q ∈ {4, 8, 16, 32, 64} at fixed hd_dim = 2048

## Outputs

- Quality table with one row per Q value containing clustered spectra ratio, incorrect clustering ratio, and entropy-based completeness
- Per-threshold metric curves across the clustering distance threshold sweep, comparable to Section 3.1 of the paper
- HyperSpec parquet clustering metadata files per Q setting

## How to apply

Run HyperSpec (python src/main.py) once per parameter setting, here for each Q in {4, 8, 16, 32, 64} with fixed --hd_dim 2048, clustering charges 2 and 3 with hierarchical clustering (complete linkage) over the bucket-wise normalized Hamming distance matrix of bit-packed hypervectors; the parquet output records bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, and is_representative per spectrum. Evaluate each Q against MS-GF+ peptide labels (charges 2 and 3) obtained from MassIVE reanalysis RMSV000000091.3 for PXD000561 raw files, sweeping the clustering distance threshold as in the paper's Section 3.1 sensitivity analysis. For each threshold and Q, compute the clustered spectra ratio, the incorrect clustering ratio (spectra clustered with a different peptide label), and entropy-based completeness. Expect quality to be relatively insensitive to Q: increasing Q from 4 to 64 reduced clustered spectra ratio and completeness (DBSCAN completeness 0.8644 → 0.8595) while slightly improving incorrect clustering ratio (1.41% → 1.29%), which justified choosing Q = 16 as the default. Assemble one row per Q with all three metrics into a single table and check trend consistency with the paper's reported sensitivity of clustering quality to hypervector generation parameters.

## Related tools

- **HyperSpec** (Produces the per-Q clustering results (parquet cluster labels) that are mapped to MS-GF+ identifications for quality scoring; invoked with --hd_dim 2048 and varying --hd_Q) — https://github.com/wh-xu/Hyper-Spec
- **Python** (Runs HyperSpec via python src/main.py and assembles the per-Q metric table from clustering output and identification labels)
- **ThermoRawFileParser** (Converts PXD000561 PRIDE raw files to MGF format as the input spectra for clustering and downstream quality evaluation)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --hd_dim 2048 --hd_Q 16 --cluster_alg hc_complete --cluster_charges 2 3 --eps=0.25 --refine
```

## Evaluation signals

- Output table contains exactly one row per Q value in {4, 8, 16, 32, 64} with all three metrics (clustered spectra ratio, incorrect clustering ratio, completeness) populated
- Trend matches the paper: incorrect clustering ratio slightly decreases with Q (e.g., DBSCAN 1.41% at Q=4 to 1.29% at Q=64) and completeness slightly decreases (0.8644 → 0.8595)
- Parquet output schema check: each line has bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative
- Only charges 2 and 3 spectra are scored, matching the MS-GF+ label set from RMSV000000091.3
- Clustering distance threshold sweep reproduces the qualitative sensitivity behavior reported in Section 3.1

## Limitations

- Clustering quality is relatively insensitive to Q, so differences across Q values are small and may be hard to distinguish from threshold-sweep noise
- Quality evaluation requires external identifications; PXD000561 labels came from a 2016/05/23 UniProtKB/Swiss-Prot human reference proteome search, so conclusions are label-set dependent
- Clustering the full PXD000561 dataset (25 million spectra, 131 GB) requires a GTX 3090 with larger GPU memory

## Evidence

- [methods] MS-GF+ identifications anchor quality evaluation: "The corresponding spectrum identifications were downloaded from MassIVE reanalysis RMSV000000091.3."
- [methods] PXD000561 used for clustering quality evaluation: "The largest data set, PXD000561,1 was used for runtime and clustering quality evaluation."
- [readme] README documents hd_dim/hd_Q defaults and parquet output schema: "--hd_dim                    The HD dimension. (default: 2048)
    --hd_Q                      The HD quantization level. (default: 16)"
- [readme] README documents the exported clustering metadata used for mapping: "records `bucket`, `precursor_charge`, `precursor_mz`, `identifier`, `scan`, `retention_time`, `cluster`, and `is_representative` information."
- [results] Sensitivity analysis framework for hypervector parameters: "as the HV dimension D increases, the incorrect clustering ratio and the clustered spectra ratio for both clustering algorithms decreased"
