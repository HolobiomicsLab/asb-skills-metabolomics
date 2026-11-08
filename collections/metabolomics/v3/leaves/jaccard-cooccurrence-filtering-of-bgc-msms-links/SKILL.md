---
name: jaccard-cooccurrence-filtering-of-bgc-msms-links
description: Use when NPOmix KNN classification has produced candidate gene cluster
  family (GCF)-to-MS/MS spectrum links and you need a post-hoc confidence filter scored
  against PoDP-validated gene-metabolite links.
when_to_use_negative:
- The query genome has no BGC similar to any BGC in the NPOmix training dataset (e.g.,
  a fungal species dissimilar to PoDP-derived training BGCs) — fingerprint columns
  will be empty and the filter yields no answer
- You need to distinguish isomers or regioselectivity of homologs — the correlation-based
  NPOmix linkage underlying the filter lacks that resolution
- Your dataset is very dissimilar to the current PoDP-derived training set — the reported
  precision may not be reproducible
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NPOmix
  role: Produces the candidate BGC-GCF/MS/MS links and fingerprint matrices that the
    Jaccard co-occurrence threshold filters
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes the pairwise BGC similarity scores (1 minus raw distance) used to
    build the BGC fingerprint vectors filtered by the threshold
  repo: https://bigscape-corason.secondarymetabolites.org
- name: k-nearest neighbor classifier
  role: Classifies GCF candidates for each MS/MS spectrum using similarity and biosynthetic
    class features before Jaccard filtering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:1f10f6fdf4685c120cc4fef7fe52c60ee8335b2293a2dee5ece52522304a83ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - NPOmix
  - k-nearest neighbor classifier
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# jaccard-cooccurrence-filtering-of-bgc-msms-links

## Summary

A confidence-filtering step for NPOmix candidate BGC–MS/MS links: compute the Jaccard co-occurrence index between thresholded BGC fingerprint vectors (maximum BiG-SCAPE similarity per genome) and MS/MS fingerprint vectors (maximum modified cosine score per sample) and retain links passing a 0.7 cutoff. Applying this filter with the similarity and biosynthetic class features enabled top-1 = 81.8% and top-10 = 92.9% precision on the PoDP validation set.

## When to use

Use when NPOmix KNN classification has produced candidate gene cluster family (GCF)-to-MS/MS spectrum links and you need a post-hoc confidence filter scored against PoDP-validated gene-metabolite links.

## When NOT to use

- The query genome has no BGC similar to any BGC in the NPOmix training dataset (e.g., a fungal species dissimilar to PoDP-derived training BGCs) — fingerprint columns will be empty and the filter yields no answer
- You need to distinguish isomers or regioselectivity of homologs — the correlation-based NPOmix linkage underlying the filter lacks that resolution
- Your dataset is very dissimilar to the current PoDP-derived training set — the reported precision may not be reproducible

## Inputs

- NPOmix candidate BGC-GCF/MS/MS link list for the PoDP validation samples
- BGC fingerprint matrix (rows = query BGCs; columns = genomes/metagenomes; values = maximum BiG-SCAPE similarity scores, 0-1)
- MS/MS fingerprint matrix (rows = query spectra; columns = paired experimental samples; values = maximum GNPS modified cosine scores)
- PoDP validated gene-metabolite links as ground truth
- NPOmix KNN predictions generated with similarity + biosynthetic class features

## Outputs

- Per-link Jaccard co-occurrence scores
- Filtered and unfiltered retained-link counts
- Top-1 and top-10 precision scores vs. PoDP ground truth (expected ~81.8% / ~92.9% at randomness = 0)
- jaccard_filter_metrics.csv

## How to apply

Load the NPOmix candidate BGC-GCF/MS/MS links predicted for the paired genome-metabolome samples, together with the BGC fingerprint vectors (row of maximum BiG-SCAPE similarity scores, 1 minus raw distance, per genome column) and the MS/MS fingerprint vectors (row of maximum GNPS modified cosine scores per sample column). For each candidate link, threshold both fingerprint vectors (BiG-SCAPE similarity below the 0.7 cutoff counts as absent) and compute the Jaccard co-occurrence index = |intersection| / |union| over the shared genome/sample columns. Retain only links whose Jaccard index passes the 0.7 co-occurrence threshold; record retained-link counts (the threshold dropped validated links from 22 to ~14) and compute top-1 and top-10 precision with and without the filter against PoDP validated links, targeting the reported 81.8%/92.9% at randomness equal to 0. Run the KNN classifier with both the similarity and biosynthetic class feature sets, since the filter's reported precision was obtained with those features. Write per-link Jaccard scores plus precision and retained-link counts for filtered vs. unfiltered settings to a metrics table (e.g., jaccard_filter_metrics.csv).

## Related tools

- **NPOmix** (Produces the candidate BGC-GCF/MS/MS links and fingerprint matrices that the Jaccard co-occurrence threshold filters) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes the pairwise BGC similarity scores (1 minus raw distance) used to build the BGC fingerprint vectors filtered by the threshold) — https://bigscape-corason.secondarymetabolites.org
- **k-nearest neighbor classifier** (Classifies GCF candidates for each MS/MS spectrum using similarity and biosynthetic class features before Jaccard filtering)

## Examples

```
from numpy import logical_and, logical_or; jac = logical_and(bgc_fp > 0.7, msms_fp > 0).sum() / logical_or(bgc_fp > 0.7, msms_fp > 0).sum(); keep = jac >= 0.7
```

## Evaluation signals

- With similarity + biosynthetic class features and the 0.7 Jaccard cutoff, top-1/top-10 precision should match the reported 81.8%/92.9% at randomness equal to 0
- 13 of 14 reference MS/MS spectra correctly labeled at k=3 on the PoDP validation reference set
- Co-occurrence threshold should reduce validated candidate links from ~22 to ~14; record retained counts per cutoff
- Thresholded fingerprints must respect the BiG-SCAPE similarity cutoff of 0.7 (scores below 0.7 treated as absent) and modified cosine scores range 0.0-1.0
- Unfiltered (no-Jaccard) baseline precision must be lower than filtered precision for the filter to demonstrate value

## Limitations

- NPOmix (correlation-based) cannot distinguish isomers or regioselectivity of homologs
- Precision scores may not be reproducible for datasets dissimilar to the PoDP-derived training set
- Metagenome quality can hinder BGC assembly, producing fragmented BGCs and fewer BGCs than expected, weakening fingerprint columns
- No RiPP-class validated metabolites exist in the PoDP training data, and paired data are almost exclusively bacterial (lack of plant data)
- New (non-PoDP) links surviving the filter still require proper structure elucidation to confirm planar (2D) and absolute (3D) chemistry

## Evidence

- [discussion] Reported precision on PoDP validation: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] BiG-SCAPE fingerprint cutoff semantics: "Similarity scores range from 0.0 to 1.0; for instance, identical BGCs have a perfect similarity score of 1.0, a score of 0.8 would represent that a homologous BGC is present in the genome"
- [readme] MS/MS fingerprint construction via modified cosine: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
- [intro] Feature-set choice affects performance: "We showed that the addition of biosynthetic class and substructure features improves the performance of our Natural Products Mixed Omics tool"
