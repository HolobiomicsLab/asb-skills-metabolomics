---
name: statistically-significant-peak-selection
description: Use when a peak-based NMR feature matrix from speaq 2.0 has been produced
  and you must isolate which grouped peaks differ significantly between conditions
  (e.g. onion-intake vs control mice) before metabolite identification.
when_to_use_negative:
- The input is already a binned/bucketed spectra matrix — binning loses per-peak information,
  which is exactly what this peak-based significance approach avoids
- The goal is raw spectra alignment or quantitation (speaq <= 1.2.3 CluPA/BW-ratio
  functionality) rather than peak-level differential analysis
- Only two clusters/peaks need comparing and no database annotation is intended —
  the HMDB submission and Jaccard scoring steps add no value
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: speaq 2.0
  role: Peak picking (wavelets), peak grouping, and feature matrix construction; its
    differential analysis identifies the significant peaks
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributed R package providing getWaveletPeaks, PeakGrouper, and BuildFeatureMatrix
    functions
  repo: https://cran.r-project.org/package=speaq
- name: HMDBsearchR
  role: Submits cluster ppm values to HMDB with +/- 0.02 ppm search tolerance for
    metabolite annotation
- name: R
  role: Implementation environment for the workflow and correlation clustering
  repo: https://cran.r-project.org/package=speaq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:ba1fca050ae5a865a28abd98598299ffa3355925fee83d0ed28373f35a7fe30c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - HMDBsearchR
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Statistically significant peak selection

## Summary

Selects statistically significant peaks from the speaq 2.0 feature/sample/peak-value matrix after differential analysis, then groups them by pairwise Pearson correlation (threshold 0.75) into clusters suitable for downstream metabolite annotation against HMDB.

## When to use

Use when a peak-based NMR feature matrix from speaq 2.0 has been produced and you must isolate which grouped peaks differ significantly between conditions (e.g. onion-intake vs control mice) before metabolite identification.

## When NOT to use

- The input is already a binned/bucketed spectra matrix — binning loses per-peak information, which is exactly what this peak-based significance approach avoids
- The goal is raw spectra alignment or quantitation (speaq <= 1.2.3 CluPA/BW-ratio functionality) rather than peak-level differential analysis
- Only two clusters/peaks need comparing and no database annotation is intended — the HMDB submission and Jaccard scoring steps add no value

## Inputs

- Raw 1D NMR spectra matrix with ppm vector and class factor (e.g. Winedata$spectra, Winedata$ppm, Winedata$wine.color)
- Significant peak table produced by the speaq 2.0 workflow (e.g. significant peaks from onion intake in mice data)

## Outputs

- Significant peak list with ppm values and group membership
- Correlation clusters of significant peaks (e.g. cluster of 4 peaks with minimal within-cluster Pearson correlation 0.75)
- metabolite_annotation_results.csv with cluster annotations, matched HMDB entries, and Jaccard scores

## How to apply

Run the speaq 2.0 workflow (wavelet peak picking via getWaveletPeaks, grouping via PeakGrouper, feature matrix construction via BuildFeatureMatrix) and apply the package's differential analysis to identify peaks with statistically significant differences between the biological groups. Compute pairwise Pearson correlations across the significant peak features and perform correlation-based clustering with a minimal within-cluster Pearson correlation of 0.75, forcing 5 clusters, so co-varying peaks likely from the same metabolite are bundled. Submit each cluster's ppm values to HMDB via HMDBsearchR with a +/- 0.02 ppm tolerance and score cluster-HMDB matches with the Jaccard index over shared peak indices. The rationale is that avoiding the binning step preserves per-peak information, so significance is called on true peaks rather than averaged bins, and clusters — not single peaks — give more robust database identifications.

## Related tools

- **speaq 2.0** (Peak picking (wavelets), peak grouping, and feature matrix construction; its differential analysis identifies the significant peaks) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributed R package providing getWaveletPeaks, PeakGrouper, and BuildFeatureMatrix functions) — https://cran.r-project.org/package=speaq
- **HMDBsearchR** (Submits cluster ppm values to HMDB with +/- 0.02 ppm search tolerance for metabolite annotation)
- **R** (Implementation environment for the workflow and correlation clustering) — https://cran.r-project.org/package=speaq

## Examples

```
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- The significant peak table must reproduce the reported Jaccard scores when annotated: 4/9 (rising to 5/9 with peak 18662) for 3-hydroxyphenylacetic acid and 1/1 for dimethyl sulfone at 3.1558 ppm
- Correlation clustering must use a minimal within-cluster Pearson correlation of 0.75 and yield 5 forced clusters over the significant peaks
- HMDB search tolerance must be set to +/- 0.02 ppm; matches outside this window are invalid
- The feature matrix must come from peak picking + grouping (not binning), i.e. a feature/sample/peak-value matrix from BuildFeatureMatrix
- The annotation table (metabolite_annotation_results.csv) must pair each cluster with its matched HMDB entry (e.g. HMDB00440) and Jaccard index

## Limitations

- The grouping step can delete one of the large peaks, which is not always mitigated by the peak filling step; the optional SilhouetR function detects this via silhouette values per group
- Spectral alignment results are not always optimal when dealing with severely shifted spectra
- The absence of standardized automated workflows means substantial user intervention can still introduce bias and lower reproducibility

## Evidence

- [abstract] starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] Compute pairwise Pearson correlations across the significant peak features and perform correlation-based clustering: [evidence span withheld — anchor status: not_found]
- [supplementary] This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group.: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
- [intro] the ppm values of the actual peaks of interest that can then be used to query a database, like HMDB: "the ppm values of the actual peaks of interest that can then be used to query a database, like HMDB [24]"
- [readme] peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector): [evidence span withheld — anchor status: not_found]
