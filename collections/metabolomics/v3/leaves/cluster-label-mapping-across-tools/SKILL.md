---
name: cluster-label-mapping-across-tools
description: Use when comparing how different spectrum clustering tools fragment the
  spectra of a specific peptide (e.g. VATVSIPR/2) — i.e. you have per-spectrum cluster
  labels from several tools plus identifications and want a per-tool unique-cluster
  count.
when_to_use_negative:
- You need clustering quality metrics such as purity/completeness across all spectra,
  not the fragmentation of a single peptide's spectra — the article measures those
  separately.
- The identifications have not been filtered at 1% FDR, since unfiltered PSMs would
  inflate the set of spectra mapped to clusters.
- You want to run a clustering tool yourself on raw data rather than harmonize pre-existing
  clustering outputs from the Zenodo deposit.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: falcon
  role: Source of clustering labels being compared; its '-1' label is treated as non-clustered
    noise in the mapping
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Comparison tool whose clustering labels are mapped; singletons treated as
    non-clustered
- name: MS-Cluster
  role: Comparison tool whose clustering labels are mapped; run at mixture probability
    0.00001
- name: msCRUSH
  role: Comparison tool whose clustering labels are mapped; run at similarity threshold
    0.8
- name: spectra-cluster
  role: Comparison tool whose clustering labels are mapped; run at threshold 0.99999
- name: Python
  role: pandas-based loading and filtering of peptide-spectrum matches and mapping
    to cluster labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:8d76b21aee7534698f336fac239acdcb253c6359eb33e88fcdef62fe59f26df9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - MS-Cluster
  - MaRaCluster
  - Python
  - falcon
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cluster-label-mapping-across-tools

## Summary

Maps peptide-spectrum matches onto cluster labels produced by multiple MS/MS spectrum clustering tools (falcon, MaRaCluster, MS-Cluster, msCRUSH, spectra-cluster), normalizes tool-specific noise/singleton representations, and counts distinct clusters for a target peptide such as VATVSIPR/2.

## When to use

Use when comparing how different spectrum clustering tools fragment the spectra of a specific peptide (e.g. VATVSIPR/2) — i.e. you have per-spectrum cluster labels from several tools plus identifications and want a per-tool unique-cluster count.

## When NOT to use

- You need clustering quality metrics such as purity/completeness across all spectra, not the fragmentation of a single peptide's spectra — the article measures those separately.
- The identifications have not been filtered at 1% FDR, since unfiltered PSMs would inflate the set of spectra mapped to clusters.
- You want to run a clustering tool yourself on raw data rather than harmonize pre-existing clustering outputs from the Zenodo deposit.

## Inputs

- Per-tool clustering result files (spectrum-to-cluster-label assignments) from Zenodo doi:10.5281/zenodo.4721496 for falcon, MaRaCluster, MS-Cluster, msCRUSH, spectra-cluster
- Peptide-spectrum matches filtered at 1% FDR from MassIVE reanalysis RMSV000000091.3 (10487235 spectrum identifications)
- Target peptide sequence and charge (e.g. VATVSIPR, charge 2, protein P00761)

## Outputs

- Tool-to-cluster-count pairs for the target peptide (e.g. vatvsipr_cluster_fragmentation.csv)
- Per-tool count of distinct non-noise clusters containing at least one target-peptide spectrum

## How to apply

Download each tool's clustering result from the Zenodo deposit (doi:10.5281/zenodo.4721496) and the peptide-spectrum matches from MassIVE reanalysis RMSV000000091.3, filtering PSMs at 1% FDR. Load the identifications with pandas and select spectra of the target peptide and charge — for VATVSIPR/2 from pig trypsin contaminant protein P00761, observed 18657 times. For each tool's clustering file, map those spectra to their cluster labels and normalize noise representations: falcon's '-1' label and singleton clusters reported by MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster are both treated as non-clustered. Use each tool's configuration yielding approximately 1% incorrectly clustered spectra (falcon cosine distance threshold 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH similarity threshold 0.8, spectra-cluster threshold 0.99999). Count distinct non-noise clusters containing at least one target-peptide spectrum per tool and write the tool-to-cluster-count pairs (e.g. to vatvsipr_cluster_fragmentation.csv); correctness is judged against the expected ordering msCRUSH (201) < falcon (269) < MS-Cluster (705) < spectra-cluster (1688) < MaRaCluster (2192).

## Related tools

- **falcon** (Source of clustering labels being compared; its '-1' label is treated as non-clustered noise in the mapping) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Comparison tool whose clustering labels are mapped; singletons treated as non-clustered)
- **MS-Cluster** (Comparison tool whose clustering labels are mapped; run at mixture probability 0.00001)
- **msCRUSH** (Comparison tool whose clustering labels are mapped; run at similarity threshold 0.8)
- **spectra-cluster** (Comparison tool whose clustering labels are mapped; run at threshold 0.99999)
- **Python** (pandas-based loading and filtering of peptide-spectrum matches and mapping to cluster labels)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- Expected per-tool counts reproduce the finding: falcon 269 and msCRUSH 201 unique clusters for VATVSIPR/2, with MS-Cluster 705, spectra-cluster 1688, MaRaCluster 2192.
- Correct ordering: msCRUSH < falcon < MS-Cluster < spectra-cluster < MaRaCluster, consistent with falcon and msCRUSH achieving highest completeness at a low number of incorrectly clustered spectra.
- falcon labels '-1' and any singleton clusters from the other four tools are excluded before counting distinct clusters.
- Clustering configs used must be the ~1% incorrectly-clustered settings (e.g. falcon cosine distance threshold 0.05) for all tools.
- Output CSV contains exactly five tool-to-cluster-count pairs.

## Limitations

- The mapping reflects cluster label fragmentation for one specific peptide in one dataset (human draft proteome, PXD000561); it does not measure global clustering quality.
- Tool-specific noise representations (falcon '-1' vs reported singletons) require explicit normalization or cross-tool counts are not comparable.
- falcon's eps parameter crucially governs cluster purity, and the scaling method can influence the cosine threshold, so counts are sensitive to clustering thresholds.

## Evidence

- [methods] All clustering results are available on Zenodo at doi:10.5281/zenodo.4721496.: "All clustering results are available on Zenodo at doi:10.5281/zenodo.4721496."
- [methods] Spectrum identifications were downloaded from MassIVE reanalysis RMSV000000091.3.: "Spectrum identifications were downloaded from MassIVE reanalysis RMSV000000091.3."
- [results] VATVSIPR/2, which is part of the pig trypsin contaminant protein P00761, is the most frequently identified peptide in the dataset (observed 18657 times): [evidence span withheld — anchor status: not_found]
- [methods] The clustering result with approximately 1% incorrectly clustered spectra was obtained with a cosine distance threshold of 0.05.: "The clustering result with approximately 1% incorrectly clustered spectra was obtained with a cosine distance threshold of 0.05."
