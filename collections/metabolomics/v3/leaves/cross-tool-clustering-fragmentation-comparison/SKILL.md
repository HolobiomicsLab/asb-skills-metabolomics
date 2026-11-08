---
name: cross-tool-clustering-fragmentation-comparison
description: 'Use when comparing multiple spectrum clustering tools'' outputs on the
  same dataset and you need a fragmentation metric: the number of unique clusters
  a chosen peptide''s spectra are split into per tool.'
when_to_use_negative:
- You want overall clustering quality metrics (purity/completeness curves over the
  full dataset) rather than a single-peptide fragmentation statistic
- Clustering results were produced with different incorrect-clustering rates (~1%
  incorrectly clustered spectra is the reference point here), making cross-tool counts
  incomparable
- The target peptide is too rare (only a handful of spectra), so cluster counts are
  dominated by singletons and not informative about completeness
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: falcon
  role: Spectrum clustering tool whose result (cosine distance threshold 0.05, label
    '-1' as noise) is compared; splits VATVSIPR/2 into 269 clusters
  repo: https://github.com/bittremieux/falcon
- name: msCRUSH
  role: Comparison tool (similarity threshold 0.8); splits VATVSIPR/2 into 201 clusters,
    the lowest fragmentation
- name: MS-Cluster
  role: Comparison tool (mixture probability 0.00001); splits VATVSIPR/2 into 705
    clusters
- name: spectra-cluster
  role: Comparison tool (threshold 0.99999); splits VATVSIPR/2 into 1688 clusters
- name: MaRaCluster
  role: Comparison tool (threshold -30.0); splits VATVSIPR/2 into 2192 clusters, the
    highest fragmentation
- name: Python (pandas)
  role: Loads the PSM identifications, filters to the target peptide, maps spectra
    to cluster labels, and counts unique non-noise clusters per tool
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:7ef6606ac0ee07c14ef01feaa9dbaa69d41af3b4bbf867e434edc81f3ad6e673
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - MS-Cluster
  - MaRaCluster
  - Python
  - Python (pandas)
  - falcon
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-tool-clustering-fragmentation-comparison

## Summary

Quantifies how many distinct non-noise clusters different MS/MS spectrum clustering tools (falcon, msCRUSH, MS-Cluster, spectra-cluster, MaRaCluster) split the spectra of a single peptide into, revealing differences in cluster completeness at a controlled ~1% incorrectly-clustered rate.

## When to use

Use when comparing multiple spectrum clustering tools' outputs on the same dataset and you need a fragmentation metric: the number of unique clusters a chosen peptide's spectra are split into per tool.

## When NOT to use

- You want overall clustering quality metrics (purity/completeness curves over the full dataset) rather than a single-peptide fragmentation statistic
- Clustering results were produced with different incorrect-clustering rates (~1% incorrectly clustered spectra is the reference point here), making cross-tool counts incomparable
- The target peptide is too rare (only a handful of spectra), so cluster counts are dominated by singletons and not informative about completeness

## Inputs

- Clustering result files for falcon, MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster from Zenodo doi:10.5281/zenodo.4721496
- Peptide-spectrum match (PSM) identifications at 1% FDR from MassIVE reanalysis RMSV000000091.3
- Identifier of the target peptide and charge state (e.g. VATVSIPR, charge 2)

## Outputs

- Per-tool count of distinct non-noise clusters containing at least one spectrum of the target peptide
- CSV of tool-to-cluster-count pairs (e.g. vatvsipr_cluster_fragmentation.csv)

## How to apply

Download the precomputed clustering results for all tools from the Zenodo deposit doi:10.5281/zenodo.4721496 and the spectrum identifications from MassIVE reanalysis RMSV000000091.3. Load the peptide-spectrum matches with pandas at 1% FDR and filter to spectra of the target peptide (here VATVSIPR, charge 2, from pig trypsin contaminant protein P00761). For each tool's clustering result, map the peptide's spectra to cluster labels and normalize noise: treat falcon's '-1' label and the singleton clusters reported by MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster as non-clustered. Use each tool's configuration yielding ~1% incorrectly clustered spectra (falcon cosine distance 0.05, MaRaCluster -30.0, MS-Cluster mixture probability 0.00001, msCRUSH similarity 0.8, spectra-cluster threshold 0.99999) and count distinct non-noise clusters containing at least one target spectrum. Fewer clusters at equal purity indicates higher completeness; expect the ordering msCRUSH < falcon < MS-Cluster < spectra-cluster < MaRaCluster for VATVSIPR/2.

## Related tools

- **falcon** (Spectrum clustering tool whose result (cosine distance threshold 0.05, label '-1' as noise) is compared; splits VATVSIPR/2 into 269 clusters) — https://github.com/bittremieux/falcon
- **msCRUSH** (Comparison tool (similarity threshold 0.8); splits VATVSIPR/2 into 201 clusters, the lowest fragmentation)
- **MS-Cluster** (Comparison tool (mixture probability 0.00001); splits VATVSIPR/2 into 705 clusters)
- **spectra-cluster** (Comparison tool (threshold 0.99999); splits VATVSIPR/2 into 1688 clusters)
- **MaRaCluster** (Comparison tool (threshold -30.0); splits VATVSIPR/2 into 2192 clusters, the highest fragmentation)
- **Python (pandas)** (Loads the PSM identifications, filters to the target peptide, maps spectra to cluster labels, and counts unique non-noise clusters per tool)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- For VATVSIPR/2 the five cluster counts match the expected ordering: msCRUSH 201 < falcon 269 < MS-Cluster 705 < spectra-cluster 1688 < MaRaCluster 2192
- Each tool's clustering configuration yields approximately 1% incorrectly clustered spectra (falcon cosine distance threshold 0.05)
- Noise handling is normalized: falcon label '-1' and singletons from the other four tools are all excluded from the distinct-cluster count
- Only spectra identified as the target peptide (1% FDR PSMs, correct charge and protein origin) contribute to the counts
- Output CSV contains exactly five tool-to-cluster-count pairs

## Limitations

- Singleton clusters and noise labels must be explicitly normalized per tool ('-1' for falcon, singletons for the others), otherwise cross-tool counts are not comparable
- The metric is indirect: it evaluates clustering on one peptide rather than a downstream task, as the paper itself notes it evaluates tools indirectly on downstream performance
- Results depend on the chosen operating point; at a low number of incorrectly clustered spectra falcon and msCRUSH lead, but MaRaCluster outperforms the other tools in completeness at slightly higher numbers

## Evidence

- [results] VATVSIPR/2, which is part of the pig trypsin contaminant protein P00761, is the most frequently identified peptide in the dataset (observed 18657 times): [evidence span withheld — anchor status: not_found]
- [results] completeness ordering at low incorrect clustering: [evidence span withheld — anchor status: not_found]
