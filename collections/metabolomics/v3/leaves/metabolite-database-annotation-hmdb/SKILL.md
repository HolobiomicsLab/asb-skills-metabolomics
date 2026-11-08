---
name: metabolite-database-annotation-hmdb
description: Use when a peak table from NMR peak-picking/grouping (e.g. speaq 2.0)
  contains statistically significant peaks whose ppm values must be mapped to metabolite
  identities in a spectral database like HMDB.
when_to_use_negative:
- Input is already a metabolite-annotated feature table (annotation already done);
  this skill targets unannotated ppm peak lists
- The raw spectra have not been processed by a peak-based pipeline such as speaq 2.0
  — HMDB queries require ppm values of actual picked peaks, not binned/bucketed spectra
- Candidate annotation for LC-MS m/z features is needed — the workflow here queries
  ppm values from 1D NMR spectra
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: speaq 2.0
  role: Upstream peak-based NMR processing (wavelet peak picking, grouping, feature
    matrix) that produces the significant peak table whose ppm values are queried
    against HMDB
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: Distributes the getWaveletPeaks/P/PeakGrouper/BuildFeatureMatrix functions
    used to generate the peak table
  repo: https://cran.r-project.org/package=speaq
- name: R
  role: Implementation environment for correlation-based clustering (Pearson threshold
    0.75) and HMDBsearchR-based database submission
- name: HMDBsearchR
  role: Submits each cluster's ppm values to HMDB with a +/- 0.02 ppm search tolerance
    to recover matching metabolite entries
- name: HMDB
  role: Reference metabolite database queried with peak ppm values for metabolite
    identification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:83b1af3002d37dda81c39ad20a0fca062d4cf1bdcf90fd15e51d2a43c28904f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - HMDB
  - HMDBsearchR
  - R
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-database-annotation-hmdb

## Summary

Annotates significant 1D NMR peaks by submitting cluster-level ppm values to HMDB (via HMDBsearchR) with a +/-0.02 ppm search tolerance, then scores cluster-HMDB matches with a Jaccard index over shared peak indices to prioritize candidate biomarkers.

## When to use

Use when a peak table from NMR peak-picking/grouping (e.g. speaq 2.0) contains statistically significant peaks whose ppm values must be mapped to metabolite identities in a spectral database like HMDB.

## When NOT to use

- Input is already a metabolite-annotated feature table (annotation already done); this skill targets unannotated ppm peak lists
- The raw spectra have not been processed by a peak-based pipeline such as speaq 2.0 — HMDB queries require ppm values of actual picked peaks, not binned/bucketed spectra
- Candidate annotation for LC-MS m/z features is needed — the workflow here queries ppm values from 1D NMR spectra

## Inputs

- Significant peak table from the speaq 2.0 workflow (peak indices, ppm values, per-sample peak intensities)
- Pearson correlation threshold (0.75) and forced number of clusters (5)
- HMDB as the target spectral/metabolite database

## Outputs

- metabolite_annotation_results.csv with cluster annotations, matched HMDB entries (e.g. HMDB00440), and Jaccard scores
- Candidate biomarker metabolite identities, e.g. 3-hydroxyphenylacetic acid (Jaccard 4/9, rising to 5/9 when peak 18662 is included) and dimethyl sulfone (Jaccard 1/1 for peak 19723 at 3.1558 ppm)

## How to apply

Start from the significant peak table produced by the speaq 2.0 workflow (wavelet peak picking via getWaveletPeaks, grouping via PeakGrouper, feature matrix via BuildFeatureMatrix). Compute pairwise Pearson correlations across the significant peak features and cluster them with a minimal within-cluster Pearson correlation threshold of 0.75, forcing a fixed number of clusters (5 in the onion-intake-in-mice case). For each cluster, submit its ppm values to HMDB using HMDBsearchR with a search tolerance of +/- 0.02 ppm, then score each cluster-HMDB match with the Jaccard index computed over shared peak indices. Write the cluster annotations, matched HMDB entries, and Jaccard scores to metabolite_annotation_results.csv; rank candidates by Jaccard score.

## Related tools

- **speaq 2.0** (Upstream peak-based NMR processing (wavelet peak picking, grouping, feature matrix) that produces the significant peak table whose ppm values are queried against HMDB) — https://github.com/beirnaert/speaq
- **speaq R package** (Distributes the getWaveletPeaks/P/PeakGrouper/BuildFeatureMatrix functions used to generate the peak table) — https://cran.r-project.org/package=speaq
- **R** (Implementation environment for correlation-based clustering (Pearson threshold 0.75) and HMDBsearchR-based database submission)
- **HMDBsearchR** (Submits each cluster's ppm values to HMDB with a +/- 0.02 ppm search tolerance to recover matching metabolite entries)
- **HMDB** (Reference metabolite database queried with peak ppm values for metabolite identification)

## Examples

```
library(speaq); peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks)
```

## Evaluation signals

- Annotation table must reproduce the reported Jaccard score of 4/9 rising to 5/9 for 3-hydroxyphenylacetic acid (HMDB00440) on the correct correlation cluster
- Annotation table must reproduce Jaccard 1/1 for dimethyl sulfone on peak 19723 at 3.1558 ppm
- Clustering must use a minimal within-cluster Pearson correlation threshold of 0.75 while forcing 5 clusters on the 4 significant onion-intake peaks
- HMDB searches must be run at a +/- 0.02 ppm tolerance; changing the tolerance should be reported as a deviation
- Output metabolite_annotation_results.csv must contain, per cluster, the matched HMDB entry identifiers and Jaccard scores over shared peak indices

## Limitations

- Jaccard scores are partial (4/9 for the top hit), so annotation is candidate-level and benefits from orthogonal confirmation against the original study's biomarkers
- The correlation clustering is sensitive to the forced cluster count (5) and the 0.75 Pearson threshold; cluster composition changes alter which ppm set is submitted
- The optional SilhouetR function is needed to detect poor groupings (deletion of large peaks in the grouping step), which would corrupt the peak indices fed to HMDB

## Evidence

- [intro] HMDB query of picked peak ppm values: "the ppm values of the actual peaks of interest that can then be used to query a database, like HMDB [24]"
- [supplementary] onion intake in mice application context: "speaq 2.0 workflow applied to the onion intake in mice data [37]."
- [readme] peak table generation functions: [evidence span withheld — anchor status: not_found]
- [abstract] peak-based pipeline overview: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] grouping quality check: "This problem is however detected by the optional SilhouetR function in speaq 2.0 which calculates the silhouette values for each group."
