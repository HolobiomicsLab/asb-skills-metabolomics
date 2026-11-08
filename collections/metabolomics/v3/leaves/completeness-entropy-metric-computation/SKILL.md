---
name: completeness-entropy-metric-computation
description: Use when clustering quality of MS/MS spectra clustering must be quantified
  as completeness from peptide-spectrum match labels, e.g. comparing DBSCAN vs hierarchical
  clustering (complete linkage) across hypervector dimensions D = 128 to 8192.
when_to_use_negative:
- Spectra lack trusted peptide identifications (e.g. no PSM reanalysis available),
  since conditional entropy given peptide assignments cannot be computed
- The goal is to score clustering purity/precision of individual clusters rather than
  label-level completeness of the partition
- Input is raw or unclustered spectra — clustering (HyperSpec) must be run first to
  produce cluster labels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: HyperSpec
  role: Produces the clustered spectra (cluster labels per MGF spectrum) whose quality
    is scored by the completeness entropy metric
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Python
  role: Implements the conditional-entropy completeness calculation over cluster labels
    joined with MS-GF+ peptide assignments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:9b304d4ae7e3e6494e75d15a36fc7edbecc93038475a62258cf085e1158eb46e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - Python
  - graphics processing units
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# completeness-entropy-metric-computation

## Summary

Quantifies clustering quality of MS/MS spectra clusters as completeness, computed as one minus the conditional entropy of the cluster distribution given peptide assignments divided by the maximum entropy reduction the peptide assignments could provide. Used in HyperSpec to compare DBSCAN versus hierarchical clustering across hypervector dimensions on labeled data.

## When to use

Use when clustering quality of MS/MS spectra clustering must be quantified as completeness from peptide-spectrum match labels, e.g. comparing DBSCAN vs hierarchical clustering (complete linkage) across hypervector dimensions D = 128 to 8192.

## When NOT to use

- Spectra lack trusted peptide identifications (e.g. no PSM reanalysis available), since conditional entropy given peptide assignments cannot be computed
- The goal is to score clustering purity/precision of individual clusters rather than label-level completeness of the partition
- Input is raw or unclustered spectra — clustering (HyperSpec) must be run first to produce cluster labels

## Inputs

- Per-spectrum cluster labels from HyperSpec (parquet/CSV with bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative)
- Peptide-spectrum match labels from MS-GF+ search against UniProtKB/Swiss-Prot human reference proteome (MassIVE RMSV000000091.3)
- Swept hyperparameter settings (HD dimension D from 128 to 8192; DBSCAN eps in 0.2-0.45; hierarchical complete linkage)

## Outputs

- Completeness value per (D, clustering algorithm) combination
- completeness_vs_hv_dimension.csv with columns D, clustering algorithm, completeness, incorrect clustering ratio, clustered spectra ratio

## How to apply

Run HyperSpec on preprocessed MGF spectra (e.g. Data set-E of PXD000561, charges 2 and 3) to obtain per-spectrum cluster labels, then join each spectrum to its peptide assignment from the MS-GF+ search of MassIVE reanalysis RMSV000000091.3 against the UniProtKB/Swiss-Prot human reference proteome. For each (D, algorithm) combination, compute completeness as one minus the conditional entropy of the cluster distribution given peptide assignments, normalized by the maximum entropy reduction the peptide assignments could provide. Repeat for each swept HD dimension D in 128-8192, clustering each bucket distance matrix once with DBSCAN (cuML, eps in the 0.2-0.45 range) and once with hierarchical clustering with complete linkage (fastcluster). Record completeness alongside the incorrect clustering ratio and clustered spectra ratio so entropy-based quality can be interpreted against how conservative each clustering is. Validation: on Data set-E, DBSCAN completeness should decrease from ~0.8979 at D=128 to ~0.8615 at D=8192 while hierarchical clustering completeness should rise from ~0.8071 to ~0.8406, and all values must lie within [0, 1].

## Related tools

- **HyperSpec** (Produces the clustered spectra (cluster labels per MGF spectrum) whose quality is scored by the completeness entropy metric) — https://github.com/wh-xu/Hyper-Spec
- **Python** (Implements the conditional-entropy completeness calculation over cluster labels joined with MS-GF+ peptide assignments)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine  # then join output labels with MS-GF+ peptides and compute completeness = 1 - H(cluster|peptide)/H_max
```

## Evaluation signals

- All completeness values lie within [0, 1] for every (D, algorithm) combination
- DBSCAN completeness decreases from ~0.8979 at D=128 toward ~0.8615 at D=8192
- Hierarchical clustering completeness increases from ~0.8071 to ~0.8406 over the same sweep
- Both algorithm traces are present for every swept D value in completeness_vs_hv_dimension.csv
- Incorrect clustering ratio and clustered spectra ratio both decrease as HV dimension D increases for both algorithms

## Limitations

- Increasing D reduces both the incorrect clustering ratio and the clustered spectra ratio, so higher-dimensional encodings keep fewer spectra clustered — completeness trends must be read together with these coverage metrics
- Metric validity depends on correct peptide labels; spectra without reliable MS-GF+ PSMs cannot contribute to the conditional entropy
- HyperSpec requires a CUDA GPU (GTX 3090 for the PXD000561-scale dataset) to reproduce the clustering labels the metric is computed on

## Evidence

- [results] DBSCAN completeness decrease with D: [evidence span withheld — anchor status: not_found]
- [results] Both ratios decrease with D: "as the HV dimension D increases, the incorrect clustering ratio and the clustered spectra ratio for both clustering algorithms decreased"
- [results] Data set-E used for quality evaluation: "Using the draft human proteome Data set-E, we examined the clustering quality using different combinations"
- [methods] Peptide label source for entropy computation: "Spectra were searched against the UniProtKB/Swiss-Prot human reference proteome (downloaded 2016/05/23)"
- [readme] Cluster label output schema: "records `bucket`, `precursor_charge`, `precursor_mz`, `identifier`, `scan`, `retention_time`, `cluster`, and `is_representative` information"
