---
name: weighted-mean-intensity-aggregation
description: Use when a RAMClustR clustering run has assigned features (MS and idMS/MS)
  to clusters and you need to convert the feature-by-sample abundance matrix into
  per-cluster spectral intensities and exportable MSP spectra.
when_to_use_negative:
- The features have not yet been clustered - aggregation requires cluster memberships
  from the RAMClustR dendrogram cut, not a raw feature table
- The input is already a per-cluster abundance matrix - re-running weighted-mean aggregation
  would double-collapse the signals
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: RAMClustR
  role: Aggregates cluster member feature intensities into weighted-mean consensus
    spectra, exports MSP files, and produces the SpecAbund dataset
  repo: https://github.com/cbroeckl/RAMClustR
- name: R
  role: Execution environment for the RAMClustR package functions that build spectra
    and write SpecAbund.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:e1e567d87325f825215d025447f59753451efd8bbdf21fbec03723b755a3c2b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - R
  - RAMClustR
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# weighted-mean-intensity-aggregation

## Summary

Condense RAMClustR cluster member feature intensities into per-cluster consensus spectral intensities by weighting each feature's abundance, producing the SpecAbund dataset and MSP spectra where more abundant features dominate the spectrum.

## When to use

Use when a RAMClustR clustering run has assigned features (MS and idMS/MS) to clusters and you need to convert the feature-by-sample abundance matrix into per-cluster spectral intensities and exportable MSP spectra.

## When NOT to use

- The features have not yet been clustered - aggregation requires cluster memberships from the RAMClustR dendrogram cut, not a raw feature table
- The input is already a per-cluster abundance matrix - re-running weighted-mean aggregation would double-collapse the signals

## Inputs

- Clustered RAMClustR object with cluster memberships and feature-by-sample abundance matrix (post-XCMS feature detection on MS and idMS/MS data)
- Feature masses and retention times for each cluster member

## Outputs

- spectra.msp - MSP-formatted file with one in-source MS and idMS/MS spectrum per cluster
- SpecAbund.csv - condensed dataset of spectral intensities, one row per cluster and one column per sample

## How to apply

Start from the clustered RAMClustR object, which holds cluster memberships over the XCMS-detected features from both MS1 and idMS/MS data. For each cluster, aggregate member feature intensities across all faahKO samples into a consensus spectrum: the peak m/z comes from each feature's mass, and each mass's intensity is a weighted mean of that feature's intensities, so the most abundant signals dominate the spectral intensity. Format the per-cluster spectrum (precursor m/z, retention time, peak m/z-intensity pairs) and write all entries into a single spectra.msp file suitable for downstream MSFinder/Sirius import. Simultaneously collapse the feature abundance matrix to the SpecAbund dataset - one spectral intensity per cluster per sample - and write it to SpecAbund.csv. Rationale: RAMClustR's unsupervised clustering groups features sharing retention time and correlational similarity, and the weighted-mean aggregation reduces quantitative analytical variation compared to using any single feature alone.

## Related tools

- **RAMClustR** (Aggregates cluster member feature intensities into weighted-mean consensus spectra, exports MSP files, and produces the SpecAbund dataset) — https://github.com/cbroeckl/RAMClustR
- **R** (Execution environment for the RAMClustR package functions that build spectra and write SpecAbund.csv)

## Examples

```
ramclustobj <- ramclustR(ms = filename, pheno_csv = pheno, st = 5, maxt = 1, blocksize = 1000)
```

## Evaluation signals

- spectra.msp parses as a valid MSP file containing one spectrum entry per cluster with m/z-intensity peak lists
- SpecAbund.csv contains exactly one intensity row per cluster and one column per faahKO sample
- Each spectral intensity equals the weighted mean of its member feature intensities, with higher-abundance features contributing proportionally more
- Number of spectra in spectra.msp equals the number of clusters in the RAMClustR object
- Spectral entries carry a precursor m/z and retention time traceable to the cluster's member features

## Limitations

- Weighted-mean aggregation assumes features derived from the same compound share approximately the same quantitative trend across samples; deviations corrupt the consensus intensity
- The condensed SpecAbund spectrum quality depends on the correctness of upstream cluster membership assignments (dendrogram cut via dynamicTreeCut)

## Evidence

- [results] mass comes from the feature mass, each mass's abundance is a weighted mean of that feature's intensities: [evidence span withheld — anchor status: not_found]
- [results] exported as msp-formatted document: [evidence span withheld — anchor status: not_found]
- [results] SpecAbund dataset condenses feature intensities into spectral intensities using a weighted mean: [evidence span withheld — anchor status: not_found]
- [readme] Investigate the deconvoluted features in the spectra folder in MSP format: [evidence span withheld — anchor status: external_doc]
