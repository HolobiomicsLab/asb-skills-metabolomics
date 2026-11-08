---
name: cluster-noise-label-normalization
description: Use when comparing cluster assignments from multiple MS/MS clustering
  tools whose noise conventions differ — e.g. falcon's '-1' label versus unlabeled
  singletons — before evaluating cluster purity or cluster size distributions.
when_to_use_negative:
- Input is a single tool's clustering result with no cross-tool comparison — the noise-relabeling
  harmonization only matters when tools use different noise conventions
- Evaluation target includes singletons or unidentified spectra as clusterable entities
  — this skill explicitly excludes them from the incorrectly-clustered fraction
- Raw spectra need to be clustered from scratch — use the falcon tool itself rather
  than this normalization skill on existing assignments
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: falcon
  role: Source of cluster assignments with explicit '-1' noise label; provides the
    reference noise convention
  repo: https://github.com/bittremieux/falcon
- name: Python 3.8
  role: pandas-based implementation of singleton relabeling and incorrectly-clustered
    fraction computation per falcon_notebooks
  repo: https://github.com/bittremieux/falcon_notebooks
- name: MaRaCluster
  role: Its singleton clusters are relabeled as noise during harmonization
- name: MS-Cluster
  role: Its singleton clusters are relabeled as noise during harmonization
- name: msCRUSH
  role: Its singleton clusters are relabeled as noise during harmonization
- name: spectra-cluster
  role: Its singleton clusters are relabeled as noise during harmonization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:9de12e7fd4a5464c4ecd649dafb15946e1e5ab57e9524d7d3b7b961faf781fd2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - MS-Cluster
  - MaRaCluster
  - Python 3.8
  - falcon
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cluster-noise-label-normalization

## Summary

Harmonizes noise/singleton labeling across spectrum clustering tool outputs (falcon, MaRaCluster, MS-Cluster, msCRUSH, spectra-cluster) so that fractions of incorrectly clustered spectra and cluster size distributions are computed on comparable, non-noise clusters only.

## When to use

Use when comparing cluster assignments from multiple MS/MS clustering tools whose noise conventions differ — e.g. falcon's '-1' label versus unlabeled singletons — before evaluating cluster purity or cluster size distributions.

## When NOT to use

- Input is a single tool's clustering result with no cross-tool comparison — the noise-relabeling harmonization only matters when tools use different noise conventions
- Evaluation target includes singletons or unidentified spectra as clusterable entities — this skill explicitly excludes them from the incorrectly-clustered fraction
- Raw spectra need to be clustered from scratch — use the falcon tool itself rather than this normalization skill on existing assignments

## Inputs

- Per-tool cluster assignment files (CSV cluster label per spectrum) from Zenodo doi:10.5281/zenodo.4721496
- MS-GF+ peptide-spectrum match identifications from MassIVE reanalysis RMSV000000091.3, filtered at 1% FDR (10487235 identifications)
- falon/falcon_notebooks evaluation code (Python/pandas)

## Outputs

- Normalized cluster label table with unified noise convention (-1) across all five tools
- Per-tool fraction of incorrectly clustered spectra (~1% expected) written to CSV
- Per-tool cluster size distributions restricted to valid clusters (>=2 spectra, charges 2 and 3)

## How to apply

Load each tool's cluster assignments (falcon cosine distance threshold 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH cosine similarity threshold 0.8, spectra-cluster threshold 0.99999, all at ~1% incorrectly clustered spectra). Retain falcon's '-1' label as noise, and relabel any singleton cluster reported by MaRaCluster, MS-Cluster, msCRUSH, or spectra-cluster as noise so all tools share a common noise convention. Restrict evaluation to spectra with precursor charges 2 and 3 and to valid clusters containing at least two spectra. Compute the fraction of incorrectly clustered spectra as the numerator over identified spectra in non-noise clusters (ignoring unidentified spectra), using the peptide label most frequent within each cluster as the cluster's reference, per the falcon_notebooks evaluation code.

## Related tools

- **falcon** (Source of cluster assignments with explicit '-1' noise label; provides the reference noise convention) — https://github.com/bittremieux/falcon
- **Python 3.8** (pandas-based implementation of singleton relabeling and incorrectly-clustered fraction computation per falcon_notebooks) — https://github.com/bittremieux/falcon_notebooks
- **MaRaCluster** (Its singleton clusters are relabeled as noise during harmonization)
- **MS-Cluster** (Its singleton clusters are relabeled as noise during harmonization)
- **msCRUSH** (Its singleton clusters are relabeled as noise during harmonization)
- **spectra-cluster** (Its singleton clusters are relabeled as noise during harmonization)

## Examples

```
counts = clusters.groupby('cluster')['spectrum'].count(); clusters.loc[clusters['cluster'].isin(counts[counts < 2].index), 'cluster'] = -1  # relabel singletons as noise
```

## Evaluation signals

- Each tool's computed fraction of incorrectly clustered spectra is approximately 1% at its specified operating point
- No valid cluster in the normalized table has fewer than 2 spectra (singletons all carry the noise label)
- falcon assignments retain '-1' as the sole noise label; other tools use the same label after relabeling
- Denominator of the incorrectly-clustered fraction equals identified spectra in non-noise clusters (unidentified spectra excluded)
- Restricted spectra all have precursor charge 2 or 3

## Limitations

- Ignoring unidentified spectra in the incorrectly-clustered fraction means purity is measured only on PSMs filtered at 1% FDR
- Chimeric spectra can still cause spectra with deviating peptide labels inside an otherwise-pure cluster; the most-frequent-peptide convention marks them incorrect but the full effect of chimeric spectra remains an open research question
- The choice of 'noise' for singletons is a convention of the evaluation, not a property reported natively by MaRaCluster, MS-Cluster, msCRUSH, or spectra-cluster

## Evidence
