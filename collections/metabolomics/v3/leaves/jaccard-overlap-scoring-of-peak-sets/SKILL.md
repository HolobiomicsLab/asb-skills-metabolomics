---
name: jaccard-overlap-scoring-of-peak-sets
description: Use when a set of significant 1D NMR peaks (ppm values) has been clustered
  by Pearson correlation and you must rank HMDB metabolite matches by how well their
  expected peak patterns overlap the cluster.
when_to_use_negative:
- The input is raw spectra rather than a peak/feature table — the speaq 2.0 peak-picking
  and grouping steps must run first
- The goal is quantitation or spectral alignment (CluPA) rather than metabolite identification
  via database matching
- Peaks are not grouped/clustered, so no peak-index set exists to compare against
  HMDB entries with the Jaccard index
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0593
tools:
- name: speaq 2.0
  role: Produces the peak-picked, grouped feature matrix whose significant peaks are
    clustered and annotated
  repo: https://github.com/beirnaert/speaq
- name: R
  role: Environment in which clustering, HMDB search, and Jaccard scoring are implemented
  repo: https://cran.r-project.org/package=speaq
- name: HMDBsearchR
  role: Queries HMDB with cluster ppm values at +/- 0.02 ppm tolerance to retrieve
    candidate metabolite matches
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:b0c55165bc8da21a5e0a62274ecd1d36e798ffebc38b7e2de044251db34ea1da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - HMDBsearchR
  - R
  - speaq 2.0
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# jaccard-overlap-scoring-of-peak-sets

## Summary

Scores candidate metabolite annotations from HMDB searches by the Jaccard index over shared peak indices between a correlation-clustered group of significant NMR peaks and the peaks of matched HMDB entries, enabling ranking of database hits for 1D NMR metabolomics.

## When to use

Use when a set of significant 1D NMR peaks (ppm values) has been clustered by Pearson correlation and you must rank HMDB metabolite matches by how well their expected peak patterns overlap the cluster.

## When NOT to use

- The input is raw spectra rather than a peak/feature table — the speaq 2.0 peak-picking and grouping steps must run first
- The goal is quantitation or spectral alignment (CluPA) rather than metabolite identification via database matching
- Peaks are not grouped/clustered, so no peak-index set exists to compare against HMDB entries with the Jaccard index

## Inputs

- Significant peak table produced by the speaq 2.0 workflow (peak features with ppm values)
- Correlation-based peak clusters (Pearson correlation threshold 0.75, forced number of clusters)
- HMDB search results from HMDBsearchR at +/- 0.02 ppm tolerance

## Outputs

- metabolite_annotation_results.csv with cluster annotations, matched HMDB entries, and Jaccard scores
- Ranked candidate biomarker metabolites (e.g. 3-hydroxyphenylacetic acid HMDB00440, dimethyl sulfone)

## How to apply

After the speaq 2.0 workflow yields a peak table, compute pairwise Pearson correlations across significant peak features and cluster them with a minimal within-cluster Pearson correlation threshold of 0.75 (forcing 5 clusters in the onion-intake-in-mice example). Submit each cluster's ppm values to HMDB via the HMDBsearchR package using a search tolerance of +/- 0.02 ppm. For each cluster-HMDB match, compute the Jaccard index over shared peak indices to score the annotation; in the onion study this recovered 3-hydroxyphenylacetic acid (HMDB00440) with a Jaccard of 4/9 (rising to 5/9 when peak 18662 was included) and dimethyl sulfone with 1/1 for peak 19723 at 3.1558 ppm. Write the cluster annotations, matched HMDB entries, and Jaccard scores to metabolite_annotation_results.csv.

## Related tools

- **speaq 2.0** (Produces the peak-picked, grouped feature matrix whose significant peaks are clustered and annotated) — https://github.com/beirnaert/speaq
- **R** (Environment in which clustering, HMDB search, and Jaccard scoring are implemented) — https://cran.r-project.org/package=speaq
- **HMDBsearchR** (Queries HMDB with cluster ppm values at +/- 0.02 ppm tolerance to retrieve candidate metabolite matches)

## Examples

```
clusters <- cutree(hclust(cor(peakTable)), k=5); hits <- HMDBsearchR::searchHMDB(ppm=cluster_ppm, tolerance=0.02); jaccard <- length(intersect(cluster_idx, hmdb_idx)) / length(union(cluster_idx, hmdb_idx))
```

## Evaluation signals

- The annotation table reproduces the reported Jaccard score of 4/9 (rising to 5/9 when peak 18662 is included) for 3-hydroxyphenylacetic acid (HMDB00440) in the correct cluster
- The annotation table reproduces the reported Jaccard score of 1/1 for dimethyl sulfone matching peak 19723 at 3.1558 ppm
- Clustering uses a minimal within-cluster Pearson correlation of 0.75 and yields the expected 5 clusters on the onion intake in mice data
- HMDB searches use a search tolerance of +/- 0.02 ppm on the cluster ppm values
- Output CSV contains cluster annotations, matched HMDB entries, and Jaccard scores for every cluster

## Limitations

- Jaccard scoring depends on the correctness of upstream peak picking and grouping; the paper notes the speaq 2.0 algorithm may delete one of the large peaks in the grouping step, and peak filling may not always mitigate missing peaks
- Spectral alignment results are not always optimal for severely shifted spectra, which can distort ppm values submitted to HMDB
- Annotation confidence is only as good as the HMDB match at +/- 0.02 ppm tolerance; partial overlaps (e.g. 4/9) still require biological confirmation against the original study's biomarkers

## Evidence

- [supplementary] Jaccard scoring over cluster-HMDB shared peaks: [evidence span withheld — anchor status: not_found]
- [supplementary] Correlation clustering threshold: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation of reproduced Jaccard scores: [evidence span withheld — anchor status: not_found]
- [intro] Database querying rationale from the article: "the ppm values of the actual peaks of interest that can then be used to query a database, like HMDB [24]"
- [supplementary] Grouping limitation: "The speaq 2.0 algorithm deletes one of the large peaks in the grouping step"
