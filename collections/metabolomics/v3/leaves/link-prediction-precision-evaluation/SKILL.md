---
name: link-prediction-precision-evaluation
description: Use when NPOmix candidate BGC-GCF/MS-MS links exist for paired genome-MS/MS
  samples and you need to quantify precision against validated PoDP links, optionally
  filtered by a Jaccard co-occurrence threshold.
when_to_use_negative:
- The query organism (e.g., a fungal species) has no BGC similar to any BGC in the
  NPOmix training dataset — output will contain no answer to evaluate
- Links involve RiPP-class metabolites — none of the currently validated metabolites
  in the PoDP database belong to the RiPP class, so precision cannot be grounded
- The goal is confirming novel (non-PoDP) links as final metabolite structures — these
  require proper elucidation (planar 2D and absolute 3D), not just classifier precision
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2258
tools:
- name: NPOmix
  role: K-nearest neighbor classifier producing the candidate BGC-GCF/MS-MS links
    and GCF labels (k=3) whose precision is evaluated
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (basis of BGC fingerprints); applied
    with --include_singletons --mibig
  repo: https://bigscape-corason.secondarymetabolites.org
- name: GNPS
  role: Provides modified cosine similarity scores between MS/MS spectra used to build
    MS/MS fingerprints, plus reference spectra
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:df798186bf310af343a9542bc12b5b54c81effa9aef49dd44e4f56206cf149ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# link-prediction-precision-evaluation

## Summary

Evaluates NPOmix BGC-to-MS/MS candidate link precision by applying a Jaccard co-occurrence threshold (0.7) between BGC and MS/MS fingerprint vectors, then scoring filtered predictions against PoDP-validated gene-metabolite links.

## When to use

Use when NPOmix candidate BGC-GCF/MS-MS links exist for paired genome-MS/MS samples and you need to quantify precision against validated PoDP links, optionally filtered by a Jaccard co-occurrence threshold.

## When NOT to use

- The query organism (e.g., a fungal species) has no BGC similar to any BGC in the NPOmix training dataset — output will contain no answer to evaluate
- Links involve RiPP-class metabolites — none of the currently validated metabolites in the PoDP database belong to the RiPP class, so precision cannot be grounded
- The goal is confirming novel (non-PoDP) links as final metabolite structures — these require proper elucidation (planar 2D and absolute 3D), not just classifier precision

## Inputs

- NPOmix candidate BGC-GCF/MS-MS links for PoDP validation samples
- BGC fingerprint vectors (maximum BiG-SCAPE similarity scores per genome column)
- MS/MS fingerprint vectors (maximum GNPS modified-cosine similarity scores per sample column)
- PoDP validated gene-metabolite links as ground truth

## Outputs

- Per-link Jaccard co-occurrence scores
- Top-1 and top-10 precision with and without the Jaccard filter
- Retained-link counts per Jaccard cutoff
- jaccard_filter_metrics.csv

## How to apply

Build BGC fingerprint vectors as rows of maximum BiG-SCAPE similarity scores (1 minus raw distance; score 1.0 = identical BGC, 0.0 or below the 0.7 similarity cutoff = absent) with one column per genome, and MS/MS fingerprint vectors as maximum modified cosine scores per experimental sample computed in GNPS. Classify each query MS/MS fingerprint with the KNN classifier (k=3) using similarity plus biosynthetic class features to obtain GCF candidate labels. For each candidate link, compute the Jaccard co-occurrence index between the thresholded BGC and MS/MS fingerprints and retain links passing the 0.7 Jaccard co-occurrence threshold, recording retained-link counts at each evaluated cutoff (validated links drop from ~22 to ~14). Score against PoDP validated gene-metabolite links as ground truth, reporting top-1 and top-10 precision with and without the filter; correct application reproduces top-1 = 81.8% and top-10 = 92.9% (13 of 14 reference MS/MS spectra labeled correctly) at randomness = 0. Write per-link Jaccard scores plus precision and retained-link counts for filtered and unfiltered settings to jaccard_filter_metrics.csv.

## Related tools

- **NPOmix** (K-nearest neighbor classifier producing the candidate BGC-GCF/MS-MS links and GCF labels (k=3) whose precision is evaluated) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarity scores (basis of BGC fingerprints); applied with --include_singletons --mibig) — https://bigscape-corason.secondarymetabolites.org
- **GNPS** (Provides modified cosine similarity scores between MS/MS spectra used to build MS/MS fingerprints, plus reference spectra)

## Examples

```
conda activate bigscape; bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- Precision matches reported top-1 = 81.8% and top-10 = 92.9% at randomness = 0 with similarity + biosynthetic class features and the Jaccard co-occurrence threshold
- At k=3, 13 of 14 reference MS/MS spectra are correctly labeled (92.9% precision)
- Applying the Jaccard cutoff of 0.7 reduces validated links from ~22 to ~14; record retained counts at each evaluated cutoff
- Filtered-vs-unfiltered comparison shows precision gain attributable to the co-occurrence threshold, not the KNN alone
- Output CSV contains per-link Jaccard scores plus precision and retained-link counts for both settings

## Limitations

- The precision score may not be reproducible for every dataset, especially if the dataset is not similar to the current training set
- Metagenome quality can hinder BGC assembly, yielding fewer and fragmented BGCs in the fingerprints
- NPOmix (a correlation-based approach) cannot distinguish isomers or regioselectivity of homologs

## Evidence

- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [intro] we deployed a K-Nearest Neighbor (KNN) algorithm that uses similarity BGC fingerprints and analogously similarity MS/MS fingerprints to classify gene cluster family: "we deployed a K-Nearest Neighbor (KNN) algorithm that uses similarity BGC fingerprints and analogously similarity MS/MS fingerprints to classify gene cluster family"
- [readme] we identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
- [readme] the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
