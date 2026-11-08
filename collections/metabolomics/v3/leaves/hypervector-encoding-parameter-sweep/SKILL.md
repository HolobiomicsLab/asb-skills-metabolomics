---
name: hypervector-encoding-parameter-sweep
description: 'Use when reproducing or extending the HyperSpec parameter sensitivity
  analysis: you need to re-encode spectra at multiple hd_dim/hd_Q values and quantify
  clustering quality (entropy-based completeness) per configuration on MS/MS data.'
when_to_use_negative:
- When the goal is a single production clustering run rather than a parameter sensitivity
  analysis — use the fixed default configuration (hd_dim 2048, hd_Q 16) directly via
  python src/main.py
- When spectra carry no peptide-level ground truth labels — the entropy-based completeness
  metric requires MS-GF+ identifications to score clustering quality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Encodes preprocessed spectra into bit-packed binary hypervectors (level-ID
    encoding) at each swept hd_dim/hd_Q configuration; invoked via python src/main.py
    with --hd_dim and --hd_Q flags
  repo: https://github.com/wh-xu/Hyper-Spec
- name: graphics processing units
  role: CUDA XOR and popc intrinsics compute the bucket-wise normalized Hamming distance
    matrices; GTX 1080Ti or 3090 class GPUs required (3090 for the 25-million-spectrum
    PXD000561 dataset)
- name: Python
  role: Runs the sweep driver, quality metric computation, and result table assembly
- name: ThermoRawFileParser
  role: Converts PXD000561 raw files downloaded from PRIDE into MGF format before
    preprocessing
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:e18a9deaab98fd5ba6ce141bb7b63785d457ee8f01a9103e3cbc8938298f65e0
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
  - graphics processing units
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# hypervector-encoding-parameter-sweep

## Summary

Sweeps HyperSpec's hyperdimensional encoding parameters — HV dimension D (128–8192) and quantization level Q (4–64) — to characterize their effect on clustering completeness, incorrect clustering ratio, and clustered spectra ratio under DBSCAN and hierarchical clustering. It establishes D=2048/Q=16 as the default configuration by trading off clustering quality against encoding cost.

## When to use

Use when reproducing or extending the HyperSpec parameter sensitivity analysis: you need to re-encode spectra at multiple hd_dim/hd_Q values and quantify clustering quality (entropy-based completeness) per configuration on MS/MS data.

## When NOT to use

- When the goal is a single production clustering run rather than a parameter sensitivity analysis — use the fixed default configuration (hd_dim 2048, hd_Q 16) directly via python src/main.py
- When spectra carry no peptide-level ground truth labels — the entropy-based completeness metric requires MS-GF+ identifications to score clustering quality

## Inputs

- Preprocessed MGF peak files from PXD000561 (ThermoRawFileParser-converted raw files, charges 2 and 3)
- MS-GF+ peptide spectrum identifications from MassIVE reanalysis RMSV000000091.3 (searched against UniProtKB/Swiss-Prot human reference proteome) as clustering quality labels

## Outputs

- completeness_vs_hv_dimension.csv with columns D, clustering algorithm, completeness, incorrect clustering ratio, clustered spectra ratio (D sweep)
- Quantization sweep table with one row per Q value in {4, 8, 16, 32, 64} and the three quality metrics (Q sweep)

## How to apply

Preprocess MGF spectra per the HyperSpec pipeline (charges 2 and 3, remove precursor and <1% base-peak peaks, drop spectra with <5 valid peaks or <250 Da mass range, keep the top-50 most intense peaks, L2-normalize intensities to [0,1]), then bucket by ascending precursor m/z. For each D in {128,...,8192} or Q in {4,8,16,32,64} at fixed hd_dim 2048, re-encode each bucket into bit-packed binary hypervectors using level-ID encoding (default reference configuration hd_dim 2048, hd_Q 16), compute the bucket-wise normalized Hamming distance matrix on GPU with CUDA XOR and popc intrinsics, and cluster each bucket with both DBSCAN (cuML, eps in the 0.2–0.45 range) and hierarchical clustering with complete linkage (fastcluster). Compute entropy-based completeness against MS-GF+ peptide labels (one minus conditional entropy of cluster distribution given peptide assignments divided by maximum entropy reduction), plus incorrect clustering ratio and clustered spectra ratio. Assemble a table with columns for D (or Q), algorithm, completeness, incorrect clustering ratio, and clustered spectra ratio.

## Related tools

- **HyperSpec** (Encodes preprocessed spectra into bit-packed binary hypervectors (level-ID encoding) at each swept hd_dim/hd_Q configuration; invoked via python src/main.py with --hd_dim and --hd_Q flags) — https://github.com/wh-xu/Hyper-Spec
- **graphics processing units** (CUDA XOR and popc intrinsics compute the bucket-wise normalized Hamming distance matrices; GTX 1080Ti or 3090 class GPUs required (3090 for the 25-million-spectrum PXD000561 dataset))
- **Python** (Runs the sweep driver, quality metric computation, and result table assembly)
- **ThermoRawFileParser** (Converts PXD000561 raw files downloaded from PRIDE into MGF format before preprocessing)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg hc_complete --cluster_charges 2 3 --eps=0.25 --hd_dim 8192 --hd_Q 16 --refine
```

## Evaluation signals

- D sweep: DBSCAN completeness decreases from ~0.8979 at D=128 to ~0.8615 at D=8192, while hierarchical clustering (complete linkage) completeness increases from ~0.8071 to ~0.8406
- D sweep: incorrect clustering ratio and clustered spectra ratio decrease for BOTH algorithms as D increases
- Q sweep: incorrect clustering ratio slightly improves (DBSCAN 1.41% → 1.29% from Q=4 to Q=64) while clustered spectra ratio and completeness decrease (DBSCAN completeness 0.8644 → 0.8595)
- All completeness values lie within [0, 1] and both algorithm traces are present for every swept D value
- Q sweep output table contains one populated row (all three metrics) per Q value in {4, 8, 16, 32, 64}

## Limitations

- Clustering quality is relatively insensitive to Q compared to D; Q=16 was chosen as the default rather than an extremum of any metric
- The sweep reflects only the PXD000561 draft human proteome Data set-E with MS-GF+ labels (UniProtKB/Swiss-Prot human reference proteome, downloaded 2016/05/23); trends may differ for other organisms or acquisition types
- PXD000561 requires a GTX 3090-class GPU with larger memory; smaller-scale sweeps can run on a GTX 1080Ti
- Clustering quality could be improved by deriving an optimized distance function for comparing spectrum HVs, which the paper lists as future work

## Evidence

- [results] D-sweep trend: "the completeness of DBSCAN decreases from 0.8979 to 0.8615, while hierarchical clustering’s completeness is improved from 0.8071 to 0.8406"
- [results] D effect on other ratios: "as the HV dimension D increases, the incorrect clustering ratio and the clustered spectra ratio for both clustering algorithms decreased"
- [methods] Bit-packed storage: "HVs are stored in a bit-packed data structure, where every 32 bits of a HV h are packed into a 32-bit integer"
- [methods] Five-step pipeline context: "The overall data processing flow of HyperSpec consists of five main steps, including spectrum preprocessing, bucket division, hyperdimensional (HD) encoding, HD distance computation, and clustering."
- [readme] CLI parameters for hd_dim and hd_Q: "--hd_dim                    The HD dimension. (default: 2048)
    --hd_Q                      The HD quantization level. (default: 16)"
- [readme] GPU requirements: "Clustering for PXD000561 dataset requires GTX 3090 with larger memory
- Clustering for other dataset with smaller scale requires GTX 1080Ti"
