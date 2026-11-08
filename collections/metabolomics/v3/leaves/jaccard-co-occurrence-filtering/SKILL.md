---
name: jaccard-co-occurrence-filtering
description: Use when a KNN classifier has returned candidate GCF labels for MS/MS
  spectra from paired genome-metabolome samples and predictions must be pruned by
  biosynthetic co-occurrence before computing precision or prioritizing links.
when_to_use_negative:
- When the dataset differs substantially from the PoDP-derived training set — the
  article warns the precision score may not be reproducible for datasets not similar
  to the current training set
- When the query organism (e.g., a fungal species) has no BGC similar to any training-set
  BGC — the co-occurrence filter cannot rescue predictions from an empty KNN output
- When candidate GCFs are legitimately absent from the sequenced genome of the matched
  sample due to fragmented or low-quality metagenome assemblies — the filter would
  wrongly discard true links
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_0091
tools:
- name: NPOmix
  role: Implements the KNN classification and applies the Jaccard co-occurrence threshold
    with similarity plus biosynthetic class features (randomness = 0)
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: scikit-learn
  role: Provides the k-nearest neighbor classifier whose majority GCF labels are filtered
    by the Jaccard co-occurrence threshold
- name: BiG-SCAPE
  role: Generates the pairwise BGC similarity scores (1 minus raw distance) underlying
    the fingerprints whose co-occurrence is tested
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:2ed4928df62427dad1508cc4ddc8b9809800800641aa796433302cc95ab2aab3
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Jaccard co-occurrence filtering

## Summary

A post-classification filtering step in the NPOmix KNN pipeline that removes predicted gene cluster family (GCF) candidates absent from the genome of the matched paired sample, boosting precision from ~81.8% to 92.9% (top-10) using a Jaccard index cutoff of 0.7.

## When to use

Use when a KNN classifier has returned candidate GCF labels for MS/MS spectra from paired genome-metabolome samples and predictions must be pruned by biosynthetic co-occurrence before computing precision or prioritizing links.

## When NOT to use

- When the dataset differs substantially from the PoDP-derived training set — the article warns the precision score may not be reproducible for datasets not similar to the current training set
- When the query organism (e.g., a fungal species) has no BGC similar to any training-set BGC — the co-occurrence filter cannot rescue predictions from an empty KNN output
- When candidate GCFs are legitimately absent from the sequenced genome of the matched sample due to fragmented or low-quality metagenome assemblies — the filter would wrongly discard true links

## Inputs

- KNN classifier output: predicted GCF labels per query MS/MS fingerprint (k nearest neighbors from the BGC fingerprint training matrix)
- BGC fingerprint training matrix (5,421 BGCs x 1,040 genomes/metagenomes, BiG-SCAPE-derived similarity scores binarized below the 0.7 similarity cutoff, with GCF label column)
- MS/MS fingerprint testing matrix (GNPS modified cosine scores of 22 reference spectra against each paired sample's pooled experimental spectra)
- Experimentally validated genome-metabolite links from PoDP/MIBiG for precision evaluation

## Outputs

- Filtered list of GCF candidates per query MS/MS spectrum (candidates absent from the matched sample's genome removed)
- Precision metrics at top-1 and top-10 (expected: 0.818 and 0.929 at k=3 with similarity + biosynthetic class features)
- Annotation rates for linked spectra (expected ~44% with threshold vs ~28% without)

## How to apply

After training the scikit-learn k-nearest neighbor classifier on the BGC fingerprint matrix (5,421 BGCs x 1,040 genomes/metagenomes from PoDP, with GCF labels) and classifying query MS/MS fingerprints, apply the Jaccard co-occurrence threshold to each prediction: compute the Jaccard index between the GCF candidate's occurrence pattern and the matched sample's genome content, and remove GCF candidates absent from the genome of the sample where the spectrum was observed. Use a Jaccard index cutoff of 0.7, with randomness set to 0, and evaluate with the similarity + biosynthetic class feature set. Compare surviving predictions against experimentally validated PoDP/MIBiG links to compute top-1 and top-10 precision; correct application should reproduce top-10 precision of 0.929 (13/14 reference spectra) and top-1 of 0.818, with validated links such as albicidin and CCMSLIB00004679300 (putative orfamide C) retained among the predictions.

## Related tools

- **NPOmix** (Implements the KNN classification and applies the Jaccard co-occurrence threshold with similarity plus biosynthetic class features (randomness = 0)) — https://github.com/tiagolbiotech/NPOmix_python
- **scikit-learn** (Provides the k-nearest neighbor classifier whose majority GCF labels are filtered by the Jaccard co-occurrence threshold)
- **BiG-SCAPE** (Generates the pairwise BGC similarity scores (1 minus raw distance) underlying the fingerprints whose co-occurrence is tested)

## Examples

```
sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/  # then run the NPOmix dereplication notebook with the Jaccard co-occurrence threshold enabled
```

## Evaluation signals

- Top-10 precision reaches 0.929 (13 of 14 reference MS/MS spectra correctly labeled) at k=3 with similarity + biosynthetic class features, randomness = 0
- Top-1 precision equals 0.818 under the same settings
- The albicidin analog link and the CCMSLIB00004679300 putative orfamide C link appear among the retained predictions
- Annotation rate for linked spectra rises from ~28% (no threshold) to ~44% (with threshold)
- The same precision values must match Fig. 3 of the paper when the sweep is written to fig3_precision_evaluation.csv

## Limitations

- Precision may not be reproducible for every dataset, especially datasets dissimilar to the PoDP-derived training set
- NPOmix is correlation-based and cannot distinguish isomers or regioselectivity of homologs, so filtered links still require structural elucidation
- Metagenome quality can hinder BGC assembly, producing fewer/fragmented BGCs that degrade co-occurrence filtering
- No RiPP-class validated metabolites exist in the PoDP training data, so filtering performance for RiPPs is untested

## Evidence

- [discussion] Precision with Jaccard threshold: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [intro] Adding class features improves performance: "We showed that the addition of biosynthetic class and substructure features improves the performance of our Natural Products Mixed Omics tool"
- [readme] Feature matrix dimensions: "for our first release, round 4, we have used 5,421 BGCs in 1,040 genomes/metagenomes"
- [readme] Similarity cutoff semantics: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [discussion] Metagenome assembly limitation: "the quality of their metagenomes can hinder BGC assembly, leading to a smaller number of BGCs than expected and many of these BGCs can be fragmented"
