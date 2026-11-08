---
name: precision-metric-computation-for-bgc-metabolite-links
description: Use when evaluating an automated BGC–metabolite linking method against
  experimentally validated PoDP/MIBiG links, to report top-1 and top-n precision on
  reference GNPS MS/MS spectra.
when_to_use_negative:
- The query organism has no BGC similar to any BGC in the NPOmix training set (e.g.,
  a fungal species dissimilar to the PoDP training genomes) — the output would contain
  no answer and precision is undefined
- The dataset differs substantially from the PoDP training set — the article warns
  the precision score may not be reproducible for every dataset
- The goal is to distinguish isomers or regioselectivity of homologs — NPOmix (a correlation-based
  approach) lacks that resolution, so precision here measures family-level linkage
  only
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: NPOmix
  role: K-nearest-neighbor classifier whose predicted GCF links for reference MS/MS
    spectra are scored against validated links
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity (similarity cutoff 0.7, --include_singletons
    --mibig) used to build the training fingerprint matrix
- name: GNPS
  role: Supplies reference MS/MS spectra and modified cosine scores for the MS/MS
    fingerprint testing matrix
- name: MIBiG
  role: Expert-annotated database of validated BGCs used as ground truth labels for
    precision computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:380a60f0a0e6a070a9da5b2546edbf179f4bb4026b3f3f40455b19a3d20a673b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - MIBiG
  - MIBiG database
  - NPOmix
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# precision-metric-computation-for-bgc-metabolite-links

## Summary

Benchmarks automated BGC–metabolite linkage predictions (e.g., NPOmix KNN output) against experimentally validated PoDP/MIBiG links by computing top-1 and top-n precision on reference GNPS MS/MS spectra, using a Jaccard co-occurrence filter.

## When to use

Use when evaluating an automated BGC–metabolite linking method against experimentally validated PoDP/MIBiG links, to report top-1 and top-n precision on reference GNPS MS/MS spectra.

## When NOT to use

- The query organism has no BGC similar to any BGC in the NPOmix training set (e.g., a fungal species dissimilar to the PoDP training genomes) — the output would contain no answer and precision is undefined
- The dataset differs substantially from the PoDP training set — the article warns the precision score may not be reproducible for every dataset
- The goal is to distinguish isomers or regioselectivity of homologs — NPOmix (a correlation-based approach) lacks that resolution, so precision here measures family-level linkage only

## Inputs

- Paired genome–MS/MS sample set from the Paired Omics Data Platform (PoDP) with 1,040 paired-sample metabolomes (GNPS job task=b915d1f78c364bf4be9900047760d95c)
- BGC fingerprints: vectors of maximum BiG-SCAPE similarity scores per genome, binarized below the 0.7 similarity cutoff
- MS/MS fingerprints: GNPS modified cosine scores of each reference spectrum (e.g., 22 GNPS reference spectra) against each paired sample's spectra
- Experimentally validated BGC–metabolite links from PoDP and MIBiG (ground truth labels)

## Outputs

- Top-1 precision score for predicted GCF links (expect 0.818 on the PoDP benchmark)
- Top-n (top-10) precision score for predicted GCF links (expect 0.929; 13 of 14 reference spectra correctly labeled at k = 3)
- Per-reference-spectrum list of predicted gene cluster family links for comparison against validated links

## How to apply

Assemble the validated ground truth: paired genome–MS/MS samples from the Paired Omics Data Platform (1,040 paired-sample metabolomes from GNPS task b915d1f78c364bf4be9900047760d95c) with BGC–metabolite links recorded in PoDP/MIBiG. Run the predictor under the fixed benchmark configuration — scikit-learn KNN with k = 3, similarity plus biosynthetic class features, Jaccard co-occurrence threshold, randomness set to 0 — using BGC fingerprints built from BiG-SCAPE similarity scores binarized at the 0.7 cutoff and MS/MS fingerprints built from GNPS modified cosine scores. For each of the 14 reference MS/MS spectra (22 candidate spectra queried), compare predicted gene cluster family (GCF) links to the experimentally validated links and compute top-1 and top-n precision (top-1 = 81.8%, top-10 = 92.9%, i.e., 13 of 14 reference spectra correctly labeled). Rationale: precision on curated validated links is the only quantitative measure of whether co-occurrence-based linkage reproduces known genome–metabolite pairs rather than producing unconfirmable novel links.

## Related tools

- **NPOmix** (K-nearest-neighbor classifier whose predicted GCF links for reference MS/MS spectra are scored against validated links) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarity (similarity cutoff 0.7, --include_singletons --mibig) used to build the training fingerprint matrix)
- **GNPS** (Supplies reference MS/MS spectra and modified cosine scores for the MS/MS fingerprint testing matrix)
- **MIBiG** (Expert-annotated database of validated BGCs used as ground truth labels for precision computation)

## Examples

```
wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1
```

## Evaluation signals

- Top-10 precision reaches 0.929 (13 of 14 reference MS/MS spectra correctly labeled) and top-1 precision is 0.818
- The albicidin analog link and the CCMSLIB00004679300 putative orfamide C link appear among the predictions
- Configuration invariants hold: randomness = 0, similarity plus biosynthetic class features enabled, Jaccard co-occurrence threshold applied, k = 3 nearest neighbors
- BGC fingerprint scores below the 0.7 similarity cutoff are binarized to absent before classifier training
- Predicted GCF links match only links recorded as experimentally validated in PoDP/MIBiG when computing precision

## Limitations

- The precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set
- No RiPP-class validated metabolites exist in the PoDP training data, so precision for RiPP links cannot be measured
- Putative links (e.g., orfamides) not present in PoDP/MIBiG require proper structure elucidation or co-elution with standards to confirm, so precision only covers validated links

## Evidence

- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [abstract] we demonstrate this approach by automatically linking 18 previously known mass spectra ... to their corresponding previously experimentally validated biosynthetic genes: [evidence span withheld — anchor status: not_found]
- [discussion] the precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set: "the precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set"
- [abstract] Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform: "Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform"
- [readme] binarizing scores below the 0.7 similarity cutoff to absent: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
