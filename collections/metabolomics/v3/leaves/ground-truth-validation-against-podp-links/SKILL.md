---
name: ground-truth-validation-against-podp-links
description: Use when NPOmix (or another co-occurrence classifier) has produced candidate
  GCF–MS/MS spectrum links and you need a quantified precision estimate against PoDP
  validated gene–metabolite links as ground truth.
when_to_use_negative:
- The candidate set contains no experimentally validated links (e.g. cryptic metabolites
  absent from GNPS and PoDP) — precision cannot be scored, only structure elucidation
  can confirm them
- Query organism has no BGC similar to the NPOmix/PoDP training set (e.g. a fungal
  species with novel BGCs), which produces no answer to validate
- The goal is distinguishing isomers or regioselectivity of homologs — NPOmix correlation-based
  links lack that resolution even when validated
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NPOmix
  role: K-nearest neighbor classifier that generates the candidate BGC–GCF/MS/MS links
    being scored against PoDP ground truth
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (1 minus raw distance) and GCF assignments
    used to build the BGC fingerprints scored against validated links
  repo: https://bigscape-corason.secondarymetabolites.org
- name: k-nearest neighbor classifier
  role: Classification algorithm (scikit-learn) that labels each query MS/MS fingerprint
    with candidate GCF labels from the k nearest BGC fingerprint neighbors
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:82d2c4f61467996c1895e24b5dfe6ba6cf0fec5159c58c9d8aedb170cbaab2b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - NPOmix
  - k-nearest neighbor classifier
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ground-truth-validation-against-podp-links

## Summary

Scores NPOmix KNN candidate BGC–MS/MS links against expert-validated gene–metabolite links from the Paired Omics Data Platform (PoDP), computing top-1 and top-10 precision with and without the Jaccard co-occurrence threshold of 0.7.

## When to use

Use when NPOmix (or another co-occurrence classifier) has produced candidate GCF–MS/MS spectrum links and you need a quantified precision estimate against PoDP validated gene–metabolite links as ground truth.

## When NOT to use

- The candidate set contains no experimentally validated links (e.g. cryptic metabolites absent from GNPS and PoDP) — precision cannot be scored, only structure elucidation can confirm them
- Query organism has no BGC similar to the NPOmix/PoDP training set (e.g. a fungal species with novel BGCs), which produces no answer to validate
- The goal is distinguishing isomers or regioselectivity of homologs — NPOmix correlation-based links lack that resolution even when validated

## Inputs

- Candidate BGC–GCF to MS/MS spectrum links predicted by the NPOmix KNN classifier (k = 3)
- PoDP validated gene–metabolite links (ground truth labels)
- BGC fingerprint vectors (maximum BiG-SCAPE similarity scores per genome column)
- MS/MS fingerprint vectors (maximum GNPS modified-cosine similarity scores per sample column)

## Outputs

- Per-link Jaccard co-occurrence scores (jaccard_filter_metrics.csv)
- Top-1 and top-10 precision for filtered and unfiltered candidate sets (randomness = 0)
- Retained-link counts per evaluated Jaccard cutoff (e.g. 22 → ~14)

## How to apply

Load the candidate BGC–GCF/MS/MS links predicted by the NPOmix k-nearest neighbor classifier (k = 3 neighbors, similarity plus biosynthetic class features), treating the PoDP validated gene–metabolite links as ground truth. Build BGC fingerprints as vectors of maximum BiG-SCAPE similarity scores per genome column (cutoff 0.7) and MS/MS fingerprints as vectors of maximum GNPS modified-cosine similarity scores per paired sample column. For each candidate link compute the Jaccard co-occurrence index between the thresholded BGC and MS/MS fingerprint vectors and apply the 0.7 cutoff to filter links, recording retained counts at each evaluated cutoff (reported: validated links drop from 22 to ~14). Compute top-1 and top-10 precision for filtered and unfiltered settings at randomness equal to 0, targeting the reported values of top-1 = 81.8% and top-10 = 92.9% (13 of 14 reference MS/MS spectra correctly labeled). Write per-link Jaccard scores plus precision and retained-link counts to jaccard_filter_metrics.csv.

## Related tools

- **NPOmix** (K-nearest neighbor classifier that generates the candidate BGC–GCF/MS/MS links being scored against PoDP ground truth) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarity scores (1 minus raw distance) and GCF assignments used to build the BGC fingerprints scored against validated links) — https://bigscape-corason.secondarymetabolites.org
- **k-nearest neighbor classifier** (Classification algorithm (scikit-learn) that labels each query MS/MS fingerprint with candidate GCF labels from the k nearest BGC fingerprint neighbors)

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git && wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1
```

## Evaluation signals

- Filtered precision matches reported values: top-1 = 81.8% and top-10 = 92.9%, both at randomness equal to 0, using similarity and biosynthetic class features with the Jaccard co-occurrence threshold
- 13 of 14 reference MS/MS spectra correctly labeled at k = 3 on the validation set (paper reports automatically linking 18 previously known mass spectra overall)
- Jaccard cutoff of 0.7 reduces validated candidate links from 22 to approximately 14; retained counts recorded per evaluated cutoff
- Every candidate link row in jaccard_filter_metrics.csv carries both a Jaccard score and filtered/unfiltered precision entries
- Ground-truth labels are restricted to PoDP validated gene–metabolite links, not self-annotated matches

## Limitations

- Precision score may not be reproducible for every dataset, especially if the dataset is not similar to the current training set
- No RiPP-class validated metabolites exist in the current PoDP training data, and paired data is almost exclusively bacterial (lacking plants)
- Metagenome quality can hinder BGC assembly, producing fragmented or fewer BGCs than expected and weakening validation coverage
- NPOmix cannot distinguish isomers or regioselectivity of homologs, capping the meaning of a 'correct' validated label

## Evidence

- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [abstract] Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform: "Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform"
- [intro] Paired Omics Data Platform (PoDP) (20), which contains paired data (genome and metabolome data of the same samples): "Paired Omics Data Platform (PoDP) (20), which contains paired data (genome and metabolome data of the same samples)"
- [readme] Many paired datasets are available at the Paired omics Data Platform [(PoDP)](https://pairedomicsdata.bioinformatics.nl): "Many paired datasets are available at the Paired omics Data Platform [(PoDP)](https://pairedomicsdata.bioinformatics.nl)"
- [abstract] automatically linking 18 previously known mass spectra ... to their corresponding previously experimentally validated biosynthetic genes: [evidence span withheld — anchor status: not_found]
