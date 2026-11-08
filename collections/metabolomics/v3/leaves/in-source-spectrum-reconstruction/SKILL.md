---
name: in-source-spectrum-reconstruction
description: Use when a RAMClustR-clustered feature table (post-XCMS MS and idMS/MS
  features with cluster memberships and a feature-by-sample abundance matrix) must
  be converted into deconvoluted spectra for spectral matching or annotation.
when_to_use_negative:
- Input is a raw XCMS peak table or xcmsObj that has not yet been clustered by RAMClustR
  — the spectra step needs cluster memberships to aggregate features
- Data come from a platform without in-source fragmentation or idMS/MS acquisition,
  so cluster members cannot be interpreted as co-eluting fragments of a single compound
  spectrum
- The goal is per-feature quantification rather than per-compound spectral intensities
  — the SpecAbund matrix deliberately condenses features into spectra and loses per-feature
  resolution
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RAMClustR
  role: Aggregates cluster member feature masses and abundances into weighted-mean
    in-source/idMS-MS spectra and writes them as MSP plus the SpecAbund matrix
  repo: https://github.com/cbroeckl/RAMClustR
- name: MSFinder
  role: Downstream consumer of the exported spectra in .mat / MSP format for spectral-matching
    annotation
- name: Sirius
  role: Downstream consumer of the exported spectra in .ms format
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:9821986bf4fdda732bec4134ad6bd449487d21e60b82b29be111929557347ec3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - MSFinder
  - R
  - RAMClustR
  - Sirius
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# in-source-spectrum-reconstruction

## Summary

Reconstructs in-source MS and idMS/MS spectra from RAMClustR feature clusters by computing weighted-mean intensities of member features, then exporting the spectra as MSP files and a condensed SpecAbund matrix of per-cluster spectral intensities per sample.

## When to use

Use when a RAMClustR-clustered feature table (post-XCMS MS and idMS/MS features with cluster memberships and a feature-by-sample abundance matrix) must be converted into deconvoluted spectra for spectral matching or annotation.

## When NOT to use

- Input is a raw XCMS peak table or xcmsObj that has not yet been clustered by RAMClustR — the spectra step needs cluster memberships to aggregate features
- Data come from a platform without in-source fragmentation or idMS/MS acquisition, so cluster members cannot be interpreted as co-eluting fragments of a single compound spectrum
- The goal is per-feature quantification rather than per-compound spectral intensities — the SpecAbund matrix deliberately condenses features into spectra and loses per-feature resolution

## Inputs

- RAMClustR ramclustObj containing cluster memberships (nfeat, ann, etc.) and the feature-by-sample abundance matrix from post-XCMS clustering of MS and idMS/MS features
- Feature retention times and m/z values carried in the ramclustObj for each cluster member
- Optionally, phenoData sample metadata for sample column naming in SpecAbund

## Outputs

- MSP-formatted spectra file (spectra.msp) with one spectrum entry per cluster, including precursor m/z, retention time, and m/z-intensity peak lists
- SpecAbund dataset (SpecAbund.csv): condensed matrix of per-cluster spectral intensities, one row per cluster and one column per sample
- R spectra folder with deconvoluted spectra in MSP format for inspection

## How to apply

For each feature cluster in the ramclustObj, aggregate the member feature masses and their feature-by-sample intensities into a consensus in-source MS and idMS/MS spectrum, using a weighted mean in which more abundant signals contribute more to the spectral intensity. Each mass in the spectrum comes from the contributing feature's m/z; precursor m/z and retention time are carried from the cluster. Write all cluster spectra to a single MSP-formatted file (spectra.msp), suitable for import into MSFinder and Sirius, and collapse the feature abundance matrix to per-cluster spectral intensities per sample, writing the condensed SpecAbund dataset (SpecAbund.csv). In the stepwise workflow this follows rc.ramclustr (and optionally rc.qc / do.findmain); with the single ramclustR call the spectra folder is produced automatically. Validate that spectra.msp parses as MSP with one spectrum entry per cluster and that SpecAbund.csv has one intensity row per cluster and one column per sample.

## Related tools

- **RAMClustR** (Aggregates cluster member feature masses and abundances into weighted-mean in-source/idMS-MS spectra and writes them as MSP plus the SpecAbund matrix) — https://github.com/cbroeckl/RAMClustR
- **MSFinder** (Downstream consumer of the exported spectra in .mat / MSP format for spectral-matching annotation)
- **Sirius** (Downstream consumer of the exported spectra in .ms format)

## Examples

```
ramclustobj <- ramclustR(ms = filename, pheno_csv = pheno, st = 5, maxt = 1, blocksize = 1000); print(ramclustobj$SpecAbund[,1:6])
```

## Evaluation signals

- spectra.msp parses as a valid MSP file and contains one spectrum entry per cluster from the ramclustObj (nfeat>0 clusters each appear once)
- Each MSP entry lists m/z-intensity pairs drawn from the cluster's member features, with intensities reproducible as weighted means of the member feature abundances
- SpecAbund.csv has exactly one intensity row per cluster and one column per sample from the faahKO/phenoData sample set
- Cluster precursor m/z and retention time in the MSP entries match the corresponding ramclustObj fields
- Inspecting ramclustobj$SpecAbund[,1:6] and ramclustobj$ann returns per-cluster intensities and annotations as documented in the README

## Limitations

- Spectral intensities are weighted means of feature intensities, so quantification inherits the reduced analytical variation reported for clusters, but per-feature detail is lost in SpecAbund
- The reconstruction assumes features derived from the same compound share approximately the same retention time and quantitative trend across samples; violations degrade the spectrum
- Feature-level noise or blank signals not removed upstream (e.g. via rc.feature.filter.blanks) will propagate into the reconstructed spectra

## Evidence

- [methods] RAMClustR uses cluster membership together with abundance values from individual features to create spectra: mass comes from the feature mass, each mass's abundance is a weighted mean of that feature's intensities.: [evidence span withheld — anchor status: not_found]
- [methods] Cluster membership is used to create a third dataset, Spec Data, which condenses feature intensities into spectral intensities using a weighted mean where more abundant signals contribute more.: [evidence span withheld — anchor status: not_found]
- [readme] The README documents that deconvoluted spectra are exported in MSP format and SpecAbund is available on the ramclustobj.: "Investigate the deconvoluted features in the `spectra` folder in MSP format
## or inspect the `ramclustobj` for feature retention times, annotations etc.
print(ramclustobj$ann)
print(ramclustobj$nfeat"
- [intro] The article states the exported spectra formats are suitable for MSFinder and Sirius import.: "The 'mat' contains spectra in .mat format suitable for import into MSFinder software.  The 'ms' contains spectra in .ms format suitable for import into Sirius software."
