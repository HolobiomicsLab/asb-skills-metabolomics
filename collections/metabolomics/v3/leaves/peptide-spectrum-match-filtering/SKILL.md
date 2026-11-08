---
name: peptide-spectrum-match-filtering
description: Use when you need to select a defined subset of identified spectra (e.g.
  one peptide/charge, such as VATVSIPR charge 2 from P00761) from MassIVE/PRIDE identifications
  before joining them to spectrum clustering results.
when_to_use_negative:
- Input is raw spectra (mzML/MGF) rather than already-searched peptide-spectrum matches
  — search and FDR filtering must precede this skill.
- The goal is clustering itself or tuning clustering thresholds (e.g. falcon's eps,
  precursor/fragment tolerances) rather than selecting identified spectra from existing
  identifications.
- Downstream evaluation of consensus spectra or cluster representatives rather than
  per-peptide cluster fragmentation analysis.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Python (pandas)
  role: Loads the PSM table and filters/regions spectra by peptide sequence and charge
    before joining to cluster labels
- name: falcon
  role: Provides one of the clustering results whose spectrum-to-cluster-label CSV
    is joined against the filtered PSMs
  repo: https://github.com/bittremieux/falcon
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:855b888958a5381449d2e0d293d73ada481398ea6018aebf104fc8978faae6ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - MaRaCluster
  - Python
  - Python (pandas)
  - falcon
  - msCRUSH
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# peptide-spectrum-match-filtering

## Summary

Filters peptide-spectrum matches (PSMs) at a 1% FDR threshold and subsets them to a specific peptide (e.g. VATVSIPR/2) so that individual spectra can be mapped to per-tool cluster labels and clustered fragmentation can be quantified.

## When to use

Use when you need to select a defined subset of identified spectra (e.g. one peptide/charge, such as VATVSIPR charge 2 from P00761) from MassIVE/PRIDE identifications before joining them to spectrum clustering results.

## When NOT to use

- Input is raw spectra (mzML/MGF) rather than already-searched peptide-spectrum matches — search and FDR filtering must precede this skill.
- The goal is clustering itself or tuning clustering thresholds (e.g. falcon's eps, precursor/fragment tolerances) rather than selecting identified spectra from existing identifications.
- Downstream evaluation of consensus spectra or cluster representatives rather than per-peptide cluster fragmentation analysis.

## Inputs

- Peptide-spectrum match identification table from MassIVE reanalysis RMSV000000091.3 (1% FDR filtered)
- Clustering result files (spectrum-to-cluster-label assignments) from the Zenodo deposit doi:10.5281/zenodo.4721496 for falcon, MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster

## Outputs

- Filtered set of spectra identified as a target peptide (e.g. VATVSIPR/2, P00761)
- Spectrum-to-cluster-label mapping per clustering tool with noise/singletons normalized as non-clustered
- Tool-to-cluster-count pairs (e.g. written to vatvsipr_cluster_fragmentation.csv)

## How to apply

Download spectrum identifications from the MassIVE reanalysis (RMSV000000091.3) and load the PSM table with pandas, filtering at 1% false discovery rate, which in the falcon paper yields 10,487,235 spectrum identifications from the Kim et al. human draft proteome dataset (PRIDE PXD000561). Subset the PSMs to the peptide of interest — here VATVSIPR at charge 2, part of the pig trypsin contaminant protein P00761, the most frequently identified peptide with 18,657 observations. For each tool's clustering result, map the filtered spectra to their cluster labels, normalizing noise representations: falcon's label '-1' and singleton clusters from MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster are both treated as non-clustered. Count distinct non-noise clusters containing at least one target-peptide spectrum per tool, using the ~1% incorrectly-clustered configurations (falcon cosine distance threshold 0.05, MaRaCluster -30.0, MS-Cluster mixture probability 0.00001, msCRUSH similarity threshold 0.8, spectra-cluster threshold 0.99999).

## Related tools

- **Python (pandas)** (Loads the PSM table and filters/regions spectra by peptide sequence and charge before joining to cluster labels)
- **falcon** (Provides one of the clustering results whose spectrum-to-cluster-label CSV is joined against the filtered PSMs) — https://github.com/bittremieux/falcon

## Examples

```
psms = pd.read_csv('identifications.tsv', sep='\t'); vatv = psms[(psms['sequence'] == 'VATVSIPR') & (psms['charge'] == 2)]
```

## Evaluation signals

- After 1% FDR filtering, the identification table should contain 10,487,235 spectrum identifications for the Kim et al. human draft proteome dataset.
- The filtered VATVSIPR charge-2 subset should correspond to the most frequently identified peptide, observed 18,657 times.
- Cluster-count ordering must place msCRUSH (201) and falcon (269) lowest, then MS-Cluster (705), spectra-cluster (1688), and MaRaCluster (2192).
- falcon label '-1' and singleton clusters from the other four tools must be excluded from distinct-cluster counts (treated as non-clustered).
- Each clustering configuration used must correspond to approximately 1% incorrectly clustered spectra as defined per tool.

## Limitations

- Noise representations differ across tools (falcon uses '-1'; others report singletons), so counts are only comparable after explicit normalization.
- Cluster counts depend on the per-tool thresholds chosen to achieve ~1% incorrectly clustered spectra; other thresholds give different fragmentation.
- The paper evaluates clustering only indirectly via identifications; downstream performance of cluster representatives was not directly evaluated.

## Evidence

- [results] VATVSIPR/2, which is part of the pig trypsin contaminant protein P00761, is the most frequently identified peptide in the dataset (observed 18657 times): [evidence span withheld — anchor status: not_found]
