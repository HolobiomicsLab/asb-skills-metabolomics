---
name: msp-spectral-file-formatting
description: Use when a non-targeted MS or idMS/MS metabolomics dataset has already
  been feature-detected and clustered (e.g. by RAMClustR post-XCMS) and you need to
  convert cluster member feature intensities into MSP spectra files and a spectral
  intensity dataset for annotation or quantitative analysis.
when_to_use_negative:
- Input is still an unclustered XCMS feature table (xcmsObj) — run the clustering
  steps (rc.ramclustr) first; there are no clusters yet to write as spectra.
- You only need raw per-feature peak intensities and no spectral matching — the feature
  table itself suffices without MSP export or SpecAbund condensation.
- Spectra are destined for formats other than MSP — for MSFinder or Sirius import,
  RAMClustR provides .mat and .ms formats instead (the article separately describes
  those exports).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RAMClustR
  role: R package that performs the feature clustering and generates the MSP-format
    spectra export plus the SpecAbund dataset; the ramclustR main function writes
    the deconvoluted cluster spectra to the spectra folder in MSP format.
  repo: https://github.com/cbroeckl/RAMClustR
- name: R
  role: Runtime environment in which the RAMClustR workflow (clustering, weighted-mean
    spectral collapse, MSP export) is executed.
- name: XCMS
  role: Upstream feature detection/alignment that produces the aligned feature table
    consumed as input to clustering and downstream MSP spectral formatting.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:4cce82dbcb734f5e3514f2ef87e225fe9d06a97bf1e9d250afd3caa3d350f723
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - R
  - RAMClustR
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# msp-spectral-file-formatting

## Summary

Format per-cluster consensus spectra (precursor m/z, retention time, and weighted-mean m/z-intensity peak lists) into MSP-format files and condense the clustered feature abundance matrix into the per-cluster per-sample SpecAbund dataset, the outputs RAMClustR produces for downstream MSFinder/Sirius spectral-matching annotation.

## When to use

Use when a non-targeted MS or idMS/MS metabolomics dataset has already been feature-detected and clustered (e.g. by RAMClustR post-XCMS) and you need to convert cluster member feature intensities into MSP spectra files and a spectral intensity dataset for annotation or quantitative analysis.

## When NOT to use

- Input is still an unclustered XCMS feature table (xcmsObj) — run the clustering steps (rc.ramclustr) first; there are no clusters yet to write as spectra.
- You only need raw per-feature peak intensities and no spectral matching — the feature table itself suffices without MSP export or SpecAbund condensation.
- Spectra are destined for formats other than MSP — for MSFinder or Sirius import, RAMClustR provides .mat and .ms formats instead (the article separately describes those exports).

## Inputs

- Clustered RAMClustR object containing cluster memberships (feature-to-cluster assignment) for MS and idMS/MS features
- Feature-by-sample abundance matrix (e.g. from an XCMS fillPeaks/xcmsObj feature table with mz_rt feature columns)
- Sample phenotype/metadata (e.g. phenoData.csv) linking samples to the abundance columns

## Outputs

- spectra.msp — a single MSP-format file with one spectrum entry per cluster, each containing precursor m/z, retention time, and weighted-mean m/z-intensity peak list
- SpecAbund.csv — condensed spectral abundance table with one row per cluster and one column per sample
- RAMClustR object fields for inspection: ramclustobj$SpecAbund, ramclustobj$nfeat, ramclustobj$ann

## How to apply

Start from the clustered RAMClustR object that links each feature cluster to its member MS and idMS/MS features and to the feature-by-sample abundance matrix. For every cluster, compute each peak's abundance as a weighted mean of the contributing feature intensities across samples, so more abundant signals contribute proportionally more to the consensus spectral intensity — this reduces quantitative analytical variation versus single-feature measures. Assemble each cluster's spectrum entry with precursor m/z, retention time, and the list of feature m/z values paired with their weighted-mean intensities, and write all entries to a single MSP-formatted file (spectra.msp). Simultaneously collapse the feature abundance matrix to one intensity per cluster per sample and write that condensed SpecAbund dataset to CSV (SpecAbund.csv). Validate by parsing spectra.msp to confirm one spectrum entry per cluster with a well-formed m/z-intensity peak list, and confirm SpecAbund.csv has one intensity row per cluster and one column per sample.

## Related tools

- **RAMClustR** (R package that performs the feature clustering and generates the MSP-format spectra export plus the SpecAbund dataset; the ramclustR main function writes the deconvoluted cluster spectra to the spectra folder in MSP format.) — https://github.com/cbroeckl/RAMClustR
- **R** (Runtime environment in which the RAMClustR workflow (clustering, weighted-mean spectral collapse, MSP export) is executed.)
- **XCMS** (Upstream feature detection/alignment that produces the aligned feature table consumed as input to clustering and downstream MSP spectral formatting.)

## Examples

```
setwd(tempdir()); ramclustobj <- ramclustR(ms = filename, pheno_csv = pheno, st = 5, maxt = 1, blocksize = 1000); print(ramclustobj$SpecAbund[,1:6])
```

## Evaluation signals

- spectra.msp parses as a valid MSP file and contains exactly one spectrum entry per detected cluster, each with an m/z-intensity peak list.
- Each spectrum entry carries the cluster's precursor m/z and retention time consistent with the clustered feature masses and retention times.
- Peak intensities in each spectrum equal the weighted mean of member feature abundances, so dominant features dominate the spectral intensity rather than a simple average.
- SpecAbund.csv has one intensity row per cluster and one column per faahKO sample, matching the cluster count and sample metadata columns.
- ramclustobj$SpecAbund[,1:6] prints per-cluster per-sample intensities and ramclustobj$nfeat reports the number of features per cluster, confirming the object carries the condensed dataset.

## Limitations

- Clustered spectra only approximate true compound spectra: cluster membership groups features derived from the same compound under the assumptions of shared retention time and correlated quantitative trends, which may not hold for every feature pair.
- Weighted-mean spectral intensities reduce quantitative analytical variation relative to single features, but the condensed SpecAbund values are consensus intensities, not individually validated feature measurements.
- The MSP export captures in-source MS and idMS/MS spectra from a single experiment; annotations derived from it inherit false positive risk for unpredictable phenomena, which the clustering approach mitigates but does not eliminate.
- The workflow assumes the feature table uses mz_rt feature column names and a first sample-name column by default; other delimiters or column orders require adjusting the featdelim and sampNameCol parameters.

## Evidence

- [methods] RAMClustR uses cluster membership together with abundance values from individual features to create spectra: mass comes from the feature mass, each mass's abundance is a weighted mean of that feature's intensities.: [evidence span withheld — anchor status: not_found]
- [methods] These spectra are exported as an msp-formatted document, and cluster membership is then used to create a third dataset, Spec Data, which condenses feature intensities into spectral intensities using a weighted mean function where more abundant signals contribute more to the spectral intensity.: [evidence span withheld — anchor status: not_found]
- [readme] print(ramclustobj$SpecAbund[,1:6]): [evidence span withheld — anchor status: external_doc]
