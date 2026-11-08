---
name: psm-fdr-identification-mapping
description: 'Use when evaluating the purity of spectrum clustering results: you have
  per-spectrum cluster labels from a clustering tool and need to attach 1%-FDR PSM
  peptide identifications (e.g., from MS-GF+ via MassIVE RMSV000000091.3) to compute
  the fraction of incorrectly clustered spectra.'
when_to_use_negative:
- The input identifications are not filtered at a controlled FDR (e.g., unfiltered
  PSMs), since the ~1% incorrectly-clustered operating points of the five tools are
  only comparable at the 1% FDR identification level.
- You need absolute clustering completeness or downstream identification sensitivity
  rather than cluster purity — this skill only labels spectra within clusters as correct/incorrect
  relative to the majority peptide.
- 'Clusters contain only unidentified spectra or singletons: the method explicitly
  ignores unidentified spectra and treats singletons as noise, so no purity signal
  can be computed for them.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Python 3.8
  role: pandas-based joining of PSM identifications to cluster assignments and computation
    of incorrectly clustered spectra fractions, per the falcon_notebooks evaluation
    code
  repo: https://github.com/bittremieux/falcon_notebooks
- name: falcon
  role: source of cluster assignments with '-1' noise labels that are retained as-is
    during the identification mapping
  repo: https://github.com/bittremieux/falcon
- name: MS-Cluster
  role: source of cluster assignments whose singleton clusters are relabeled as noise
    before mapping identifications
- name: MaRaCluster
  role: source of cluster assignments whose singleton clusters are relabeled as noise
    before mapping identifications
- name: msCRUSH
  role: source of cluster assignments whose singleton clusters are relabeled as noise
    before mapping identifications
- name: spectra-cluster
  role: source of cluster assignments whose singleton clusters are relabeled as noise
    before mapping identifications
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:974085b0a6cb93aef932c89a7f1bffba0d1d4d3b4f76be7aa590b297d5d1b344
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon_notebooks
  tools:
  - MS-Cluster
  - MaRaCluster
  - Python 3.8
  - falcon
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# psm-fdr-identification-mapping

## Summary

Maps peptide-spectrum match (PSM) identifications, filtered at 1% FDR, onto spectrum clustering results so that each cluster can be assigned a majority peptide label and member spectra with deviating labels can be counted as incorrectly clustered. This is the identification-mapping step underlying the cluster purity and cluster-size-distribution comparisons of falcon, MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster.

## When to use

Use when evaluating the purity of spectrum clustering results: you have per-spectrum cluster labels from a clustering tool and need to attach 1%-FDR PSM peptide identifications (e.g., from MS-GF+ via MassIVE RMSV000000091.3) to compute the fraction of incorrectly clustered spectra.

## When NOT to use

- The input identifications are not filtered at a controlled FDR (e.g., unfiltered PSMs), since the ~1% incorrectly-clustered operating points of the five tools are only comparable at the 1% FDR identification level.
- You need absolute clustering completeness or downstream identification sensitivity rather than cluster purity — this skill only labels spectra within clusters as correct/incorrect relative to the majority peptide.
- Clusters contain only unidentified spectra or singletons: the method explicitly ignores unidentified spectra and treats singletons as noise, so no purity signal can be computed for them.

## Inputs

- MS-GF+ peptide-spectrum match identifications filtered at 1% FDR (MassIVE reanalysis RMSV000000091.3)
- Per-spectrum cluster assignments (CSV cluster label files) from falcon, MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster (Zenodo doi:10.5281/zenodo.4721496)
- Spectrum metadata: precursor charge and precursor mass per spectrum

## Outputs

- Per-cluster majority peptide labels with per-spectrum correct/incorrect flags
- Fraction of incorrectly clustered spectra per tool (CSV summary table, target ≈ 1% per tool)
- Cluster size distributions restricted to valid clusters (≥ 2 spectra) and charges 2–3

## How to apply

Retrieve the MS-GF+ peptide-spectrum match identifications from the MassIVE reanalysis RMSV000000091.3, filtered at 1% false discovery rate, which yields 10487235 spectrum identifications for the human draft proteome dataset (PRIDE PXD000561, 25 million spectra). Load each clustering tool's per-spectrum cluster assignments and join the PSM identifications to spectra by spectrum identifier using Python/pandas per the falcon_notebooks evaluation code. Normalize noise handling first: retain falcon's '-1' noise label and relabel singleton clusters reported by MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster as noise, then restrict to precursor charges 2 and 3 and to valid clusters containing at least two spectra. For each cluster determine the most frequent peptide label, mark member spectra with deviating labels as incorrectly clustered while ignoring unidentified spectra, and compute the fraction of incorrectly clustered spectra as the numerator over identified spectra in non-noise clusters; verify each tool's value is approximately 1% before comparing cluster size distributions.

## Related tools

- **Python 3.8** (pandas-based joining of PSM identifications to cluster assignments and computation of incorrectly clustered spectra fractions, per the falcon_notebooks evaluation code) — https://github.com/bittremieux/falcon_notebooks
- **falcon** (source of cluster assignments with '-1' noise labels that are retained as-is during the identification mapping) — https://github.com/bittremieux/falcon
- **MS-Cluster** (source of cluster assignments whose singleton clusters are relabeled as noise before mapping identifications)
- **MaRaCluster** (source of cluster assignments whose singleton clusters are relabeled as noise before mapping identifications)
- **msCRUSH** (source of cluster assignments whose singleton clusters are relabeled as noise before mapping identifications)
- **spectra-cluster** (source of cluster assignments whose singleton clusters are relabeled as noise before mapping identifications)

## Examples

```
import pandas as pd; clusters = pd.read_csv('falcon.csv', names=['spectrum', 'cluster']); psms = pd.read_csv('msgf_plus_1pct_fdr.tsv', sep='\t'); joined = clusters.merge(psms, on='spectrum')
```

## Evaluation signals

- The PSM set filtered at 1% FDR must yield 10487235 spectrum identifications for the human draft proteome dataset.
- Each tool's computed fraction of incorrectly clustered spectra must be approximately 1% at its designated operating point (falcon cosine distance threshold 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH cosine similarity threshold 0.8, spectra-cluster threshold 0.99999).
- Only spectra with precursor charges 2 and 3 and clusters of at least two spectra should be included; singletons (except falcon's '-1' noise label) must be excluded as noise.
- Spectra without peptide identifications must be excluded from both numerator and denominator of the incorrectly clustered fraction.
- The resulting summary table must match the published table 1 values within tolerance when reproduced via the falcon_notebooks evaluation code.

## Limitations

- Incorrectly clustered spectra are defined relative to the most frequent peptide label per cluster, ignoring unidentified spectra; chimeric spectra and cluster-representative formation effects on downstream identifications are not captured (the article lists chimeric spectra effects as an open research question).
- The ~1% operating points are tool-specific parameter choices; comparisons at other thresholds or FDR levels are not supported by this mapping procedure.
- Performance is evaluated indirectly via peptide labels rather than by directly assessing downstream task performance, as noted in the article's discussion.

## Evidence
