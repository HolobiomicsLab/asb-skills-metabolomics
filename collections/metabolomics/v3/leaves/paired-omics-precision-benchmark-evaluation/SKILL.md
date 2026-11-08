---
name: paired-omics-precision-benchmark-evaluation
description: Use when NPOmix KNN predictions (GCF labels assigned to query MS/MS spectra)
  must be scored against validated BGC-metabolite links from MIBiG/PoDP to report
  top-1 and top-10 precision.
when_to_use_negative:
- New links that are not experimentally validated in MIBiG/PoDP — these require proper
  structure elucidation and cannot serve as ground truth for precision scoring
- Datasets dissimilar to the PoDP training set (e.g. fungal species with no similar
  BGCs in the training data) — precision is not reproducible there and output may
  be empty
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: NPOmix
  role: KNN classifier producing the ranked GCF predictions that are evaluated against
    validated links
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: scikit-learn
  role: K-nearest-neighbor classifier (k=3) with appended biosynthetic class features
    used to generate predictions
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity (1 minus raw distance) used to build the
    training fingerprint matrix being benchmarked
- name: MIBiG database
  role: Source of expert-annotated validated BGC records serving as ground truth
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:198ce049523c7ec410447846eef17f86a3004061cbe9523c7d0f7e360057d659
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - MIBiG database
  - NPOmix
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# paired-omics-precision-benchmark-evaluation

## Summary

Benchmark NPOmix KNN BGC-metabolite link predictions against curated MIBiG/PoDP validated links by computing top-1 and top-10 precision (expected 81.8% and 92.9% with similarity + biosynthetic class features and the Jaccard co-occurrence threshold of 0.7).

## When to use

Use when NPOmix KNN predictions (GCF labels assigned to query MS/MS spectra) must be scored against validated BGC-metabolite links from MIBiG/PoDP to report top-1 and top-10 precision.

## When NOT to use

- New links that are not experimentally validated in MIBiG/PoDP — these require proper structure elucidation and cannot serve as ground truth for precision scoring
- Datasets dissimilar to the PoDP training set (e.g. fungal species with no similar BGCs in the training data) — precision is not reproducible there and output may be empty

## Inputs

- NPOmix KNN prediction output: ranked GCF label candidates for each of the 15 query reference MS/MS spectra
- Curated validated BGC-metabolite links from MIBiG/PoDP (ground-truth label set)
- BGC fingerprint training matrix (5,421 BGCs x 1,040 genomes, max BiG-SCAPE similarity, cutoff 0.7, plus GCF label column and biosynthetic class features)
- MS/MS fingerprint testing matrix (15 reference spectra x 1,040 paired samples, GNPS modified cosine scores)

## Outputs

- npomix_precision_results.csv with top-1 precision (expected 81.8%) and top-10 precision (expected 92.9%)
- Per-spectrum correct/incorrect label assignments at top-1, top-3, and top-10

## How to apply

Run the NPOmix pipeline (BiG-SCAPE similarity table from 5,421 BGCs across 1,040 genomes/metagenomes with similarity cutoff 0.7; KNN with k=3 and appended biosynthetic class features; GNPS modified cosine MS/MS fingerprints for 15 reference spectra), then compare each predicted GCF label set against the curated known BGC-metabolite links from MIBiG/PoDP. Count a hit when the correct GCF appears within the top-1 or top-10 ranked predictions, allowing repeated GCF labels from the k=3 neighbors. Record both precision scores (top-1 expected 81.8%, top-10 expected 92.9%) in npomix_precision_results.csv. Judge correctness by reproducing the finding that 13 of 14 reference MS/MS spectra were correctly labeled at k=3 (top-3) under the Jaccard co-occurrence threshold of 0.7.

## Related tools

- **NPOmix** (KNN classifier producing the ranked GCF predictions that are evaluated against validated links) — https://github.com/tiagolbiotech/NPOmix_python
- **scikit-learn** (K-nearest-neighbor classifier (k=3) with appended biosynthetic class features used to generate predictions)
- **BiG-SCAPE** (Computes pairwise BGC similarity (1 minus raw distance) used to build the training fingerprint matrix being benchmarked)
- **MIBiG database** (Source of expert-annotated validated BGC records serving as ground truth)

## Examples

```
sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/
```

## Evaluation signals

- Top-10 precision reproduces 92.9% (13 of 14 reference MS/MS spectra correctly labeled at k=3) with similarity + biosynthetic class features and the Jaccard co-occurrence threshold of 0.7
- Top-1 precision reproduces 81.8% with randomness equal to 0 under the same feature set and threshold
- npomix_precision_results.csv exists and records both top-1 and top-10 precision values
- GCF label repetitions from the k=3 nearest neighbors are retained in the candidate list before threshold filtering

## Limitations

- The precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set
- NPOmix does not have the resolution to distinguish between isomers and regioselectivity of the various homologs
- No RiPP-class validated metabolites exist in the PoDP training data, so performance for that biosynthetic class cannot be benchmarked

## Evidence

- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [results] With the co-occurrence threshold, 13 of 14 reference MS/MS spectra were correctly labeled at k=3 (top-3), yielding 92.9% precision using similarity and biosynthetic class features.: [evidence span withheld — anchor status: not_found]
- [readme] This matrix can be fed to the k-nearest neighbor (KNN) algorithm in order to train it with the genomic data.: "This matrix can be fed to the k-nearest neighbor (KNN) algorithm in order to train it with the genomic data."
- [discussion] the precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set: "the precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set"
