---
name: cross-tool-clustering-benchmarking
description: Use when you have cluster assignments from multiple spectrum clustering
  tools plus peptide-spectrum match identifications, and need to compare clustering
  purity/completeness or cluster size distributions at equivalent error rates.
when_to_use_negative:
- You only have a single tool's clustering output with no peptide-spectrum match identifications
  to serve as ground truth labels
- Your goal is producing consensus/meta-clusters from multiple tools rather than benchmarking
  the tools against each other
- Your spectra fall outside the evaluation scope (e.g. precursor charges other than
  2 and 3, or metabolomics/top-down data where the noise and singleton conventions
  differ)
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool under benchmark; its '-1' noise label is retained
    and its cosine distance threshold 0.05 gives the ~1% incorrectly-clustered operating
    point
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Benchmarked tool; operating point set at clustering threshold -30.0, singletons
    relabeled as noise
- name: MS-Cluster
  role: Benchmarked tool; operating point set at mixture probability 0.00001, singletons
    relabeled as noise
- name: msCRUSH
  role: Benchmarked tool; operating point set at cosine similarity threshold 0.8,
    singletons relabeled as noise
- name: spectra-cluster
  role: Benchmarked tool; operating point set at clustering threshold 0.99999, singletons
    relabeled as noise
- name: Python 3.8
  role: Runs the falcon_notebooks evaluation code (pandas) that computes incorrectly-clustered
    fractions and cluster size distributions
  repo: https://github.com/bittremieux/falcon_notebooks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:e50ee36aa1b9b50577f33f1fb2f4ee50eb4fa283a3c30e4f2c71c918691a6ab8
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-tool-clustering-benchmarking

## Summary

Benchmark MS/MS spectrum clustering tools by computing the fraction of incorrectly clustered spectra at a matched ~1% error operating point and comparing the resulting cluster size distributions, using the falcon_notebooks evaluation code. This makes clustering quality comparable across tools that use different similarity measures and thresholds.

## When to use

Use when you have cluster assignments from multiple spectrum clustering tools plus peptide-spectrum match identifications, and need to compare clustering purity/completeness or cluster size distributions at equivalent error rates.

## When NOT to use

- You only have a single tool's clustering output with no peptide-spectrum match identifications to serve as ground truth labels
- Your goal is producing consensus/meta-clusters from multiple tools rather than benchmarking the tools against each other
- Your spectra fall outside the evaluation scope (e.g. precursor charges other than 2 and 3, or metabolomics/top-down data where the noise and singleton conventions differ)

## Inputs

- Clustering result files (spectrum-to-cluster assignments) for falcon, MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster at their ~1% incorrectly-clustered operating points (deposited at Zenodo doi:10.5281/zenodo.4721496)
- MS-GF+ peptide-spectrum match identifications from MassIVE reanalysis RMSV000000091.3, filtered at 1% FDR (10487235 spectrum identifications)
- Spectrum metadata including precursor charge (restricted to charges 2 and 3)

## Outputs

- CSV summary table of the fraction of incorrectly clustered spectra per tool (each ≈1%)
- Per-tool cluster size distributions (e.g. counts of clusters in 2–100, 100–1000, 1000–10000 spectra ranges)
- Per-spectrum correctness labels (most-frequent-peptide vs. deviating peptide label within each cluster)

## How to apply

Load each tool's clustering output at its ~1% incorrectly-clustered operating point: falcon at cosine distance threshold 0.05, MaRaCluster at clustering threshold -30.0, MS-Cluster at mixture probability 0.00001, msCRUSH at cosine similarity threshold 0.8, and spectra-cluster at clustering threshold 0.99999. Normalize noise handling across tools: keep falcon's '-1' noise label and relabel singleton clusters from MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster as noise. Restrict the evaluation to spectra with precursor charges 2 and 3 and to valid clusters of at least two spectra. For each cluster, take the most frequent peptide label as the cluster identity and mark spectra with deviating labels as incorrectly clustered, ignoring unidentified spectra (Python/pandas per the falcon_notebooks code). Compute the fraction incorrectly clustered as the numerator over identified spectra in non-noise clusters, write the summary table to CSV, and verify each tool's value is approximately 1%; then compute per-cluster size histograms (e.g. 2–100, 100–1000, 1000–10000 spectra bins) to compare cluster size distributions.

## Related tools

- **falcon** (Spectrum clustering tool under benchmark; its '-1' noise label is retained and its cosine distance threshold 0.05 gives the ~1% incorrectly-clustered operating point) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Benchmarked tool; operating point set at clustering threshold -30.0, singletons relabeled as noise)
- **MS-Cluster** (Benchmarked tool; operating point set at mixture probability 0.00001, singletons relabeled as noise)
- **msCRUSH** (Benchmarked tool; operating point set at cosine similarity threshold 0.8, singletons relabeled as noise)
- **spectra-cluster** (Benchmarked tool; operating point set at clustering threshold 0.99999, singletons relabeled as noise)
- **Python 3.8** (Runs the falcon_notebooks evaluation code (pandas) that computes incorrectly-clustered fractions and cluster size distributions) — https://github.com/bittremieux/falcon_notebooks

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- Each tool's computed fraction of incorrectly clustered spectra is approximately 1% (the matched operating point)
- All clusters counted have size ≥ 2 (singletons relabeled as noise; falcon's '-1' label treated as noise)
- The denominator of the fraction is the number of identified spectra in non-noise clusters (unidentified spectra excluded)
- Each cluster's identity is assigned as its most frequent peptide label; spectra with deviating labels are the numerator
- Cluster size distributions reproduce the reported pattern: falcon, msCRUSH, and to a lesser extent MS-Cluster produce large clusters of 1000–10000 spectra, while MaRaCluster and spectra-cluster predominantly produce clusters of at most 100 spectra

## Limitations

- Performance is evaluated only at the spectrum-clustering level, not on downstream tasks; the article notes it evaluates tools 'rather than indirectly evaluating their performance on a downstream task'
- The strategy used to form cluster representatives (medoid selection vs. consensus spectra) and its effect on downstream applications was not evaluated
- Whether alternative similarity measures (e.g. spectra-cluster's probabilistic score, MaRaCluster's fragment rarity score, or machine-learned similarity measures) can more sensitively capture spectrum similarity remains unestablished

## Evidence
