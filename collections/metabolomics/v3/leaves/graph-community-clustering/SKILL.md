---
name: graph-community-clustering
description: Use when you have peak tables from SAND-decomposed 1D NMR spectra across
  many samples (e.g. 50 simulated or experimental spectra) and need to group co-occurring
  peaks into compound-level clusters for automated annotation.
when_to_use_negative:
- The input is already an annotated feature/compound table — clustering would only
  re-derive known assignments
- The spectra are multidimensional or contain nonexponential lineshapes — SAND is
  currently tuned for high-resolution 1D metabolomics spectra and does not yet handle
  these cases
- You have only one spectrum — cross-sample peak correlations require a series of
  spectra with varying compound concentrations to be informative
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: SAND
  role: Decomposes each 1D NMR spectrum into a peak table whose peaks are then correlated
    and clustered via the network
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Current implementation language of SAND's decomposition that feeds the correlation
    network
- name: NMRbox
  role: Cloud environment from which SAND decomposition (and hence the clustering
    workflow) is initiated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:c4ad6efe6005f11962f9ca3f61f497665305aff1f54b10efaafa42d6ee0e9799
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# graph-community-clustering

## Summary

Builds a peak-correlation network across a spectral series and partitions it into communities so that peaks originating from the same compound co-cluster, enabling automated annotation of SAND-decomposed 1D NMR peaks. Validated on simulated 18-compound-group mixtures with a mean single-cluster recovery ratio of 74%.

## When to use

Use when you have peak tables from SAND-decomposed 1D NMR spectra across many samples (e.g. 50 simulated or experimental spectra) and need to group co-occurring peaks into compound-level clusters for automated annotation.

## When NOT to use

- The input is already an annotated feature/compound table — clustering would only re-derive known assignments
- The spectra are multidimensional or contain nonexponential lineshapes — SAND is currently tuned for high-resolution 1D metabolomics spectra and does not yet handle these cases
- You have only one spectrum — cross-sample peak correlations require a series of spectra with varying compound concentrations to be informative

## Inputs

- Peak tables from SAND decomposition of a series of 1D 1H NMR spectra (e.g. 50 spectra from 18 compound groups with varying concentrations)
- Pairwise peak-correlation matrix computed across samples at paper-specified thresholds
- Optional: ground-truth peak-to-compound assignments for validating cluster recovery

## Outputs

- Peak-correlation network of detected peaks
- Community (cluster) assignments mapping peaks to compound groups
- Per-compound single-cluster recovery ratios (e.g. peak_recovery_ratios.csv)

## How to apply

First decompose each spectrum into a peak table using SAND's hybrid Markov chain Monte Carlo optimization with time- and frequency-domain subsampling. Then compute pairwise correlations of every detected peak across the sample series (e.g. 50 samples where compound concentrations vary from maximum down to 20%) and build the peak-correlation network at the thresholds stated in the paper. Partition the network with community detection and assign resulting clusters to known compounds (e.g. the 18 ground-truth compound groups). For each compound, compute the fraction of its ground-truth peaks that land in a single cluster and write per-compound recovery ratios (e.g. to peak_recovery_ratios.csv); the reported benchmark is >60% single-cluster recovery for most compounds and a mean recovery ratio of 74%.

## Related tools

- **SAND** (Decomposes each 1D NMR spectrum into a peak table whose peaks are then correlated and clustered via the network) — https://github.com/edisonomics/SAND
- **MATLAB** (Current implementation language of SAND's decomposition that feeds the correlation network)
- **NMRbox** (Cloud environment from which SAND decomposition (and hence the clustering workflow) is initiated)

## Evaluation signals

- Mean single-cluster recovery ratio across compounds should reproduce the reported ~74%
- For most compounds, >60% of ground-truth peaks land in one cluster
- Clusters from community detection should map one-to-one onto the 18 ground-truth compound groups in the simulated mixture
- Per-compound recovery ratios written to peak_recovery_ratios.csv should be complete (one row per compound) and fall within the reported 60–74% range for most compounds

## Limitations

- Recovery is partial: on average only 74% of peaks per compound are recovered in single clusters, so some peaks of a compound fall outside its main cluster
- Depends on SAND's decomposition quality; the current MATLAB implementation is slow and not readily parallelizable
- SAND is currently tuned for high-resolution metabolomics spectra; other spectra types (nonexponential signals, multidimensional data) are not yet accommodated

## Evidence

- [abstract] on average, 74% of peaks for each compound can be recovered in single clusters.: "on average, 74% of peaks for each compound can be recovered in single clusters."
- [results] In simulated mixtures of 50 spectra with 18 compound groups, most peaks of each compound were automatically decomposed, correlated, and clustered, with more than 60% of ground-truth peaks recovered in one cluster for most compounds (mean recovery ratio of 74%).: [evidence span withheld — anchor status: not_found]
- [readme] SAND automatic process and deconvolute 1D NMR spectra into peak tables. Similar workflow works for simulated and expermental datasets.: "SAND automatic process and deconvolute 1D NMR spectra into peak tables."
- [abstract] The SAND approach uses hybrid optimization with Markov chain Monte Carlo methods, employing subsampling in both time and frequency domains.: "The SAND approach uses hybrid optimization with Markov chain Monte Carlo methods, employing subsampling in both time and frequency domains."
