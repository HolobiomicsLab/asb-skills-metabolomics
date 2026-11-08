---
name: jaccard-cooccurrence-link-validation
description: Use when NPOmix KNN votes have produced candidate GCF assignments for
  query MS/MS spectra and you must score link confidence (e.g. top-1 precision, Jaccard
  co-occurrence threshold) before accepting a spectrum-to-GCF link.
when_to_use_negative:
- The query organism has no BGC similar to any BGC in the NPOmix training dataset
  (e.g. a fungal species) — NPOmix returns no answer, so there is nothing to validate.
- The goal is to distinguish isomers or regioselectivity of homologs — NPOmix, a correlation-based
  approach, lacks that resolution, so Jaccard co-occurrence cannot certify isomer
  identity.
- The link will be reported as a confirmed metabolite structure without further work
  — unvalidated links still require proper elucidation to confirm the metabolite's
  planar (2D) and absolute (3D) chemical structure.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: NPOmix
  role: KNN classifier whose voted GCF labels are compared to expected labels via
    Jaccard co-occurrence to validate links
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarities (1 − raw distance, 0.7 cutoff) that define
    the GCF label space against which predictions are validated
  repo: https://bigscape-corason.secondarymetabolites.org
- name: GNPS
  role: Provides modified cosine pairwise scoring used to build the MS/MS similarity
    fingerprints whose GCF assignments are validated
  repo: https://gnps.ucsd.edu
- name: antiSMASH
  role: Labels candidate BGCs (e.g. as siderophores) providing the expected labels
    for the Jaccard co-occurrence check
  repo: https://antismash.secondarymetabolites.org/#!/start
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:f7fdf550add68245ae454252b49048dddcc76a3af0f3f133f29499cfe7fdca1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Jaccard co-occurrence link validation

## Summary

Validates putative BGC-to-MS/MS links produced by NPOmix by measuring Jaccard co-occurrence between the GCF labels voted by the k-nearest-neighbor classifier and the expected (known or predicted) GCF labels, e.g. the 100% co-occurrence check for the m/z 780 siderophore link.

## When to use

Use when NPOmix KNN votes have produced candidate GCF assignments for query MS/MS spectra and you must score link confidence (e.g. top-1 precision, Jaccard co-occurrence threshold) before accepting a spectrum-to-GCF link.

## When NOT to use

- The query organism has no BGC similar to any BGC in the NPOmix training dataset (e.g. a fungal species) — NPOmix returns no answer, so there is nothing to validate.
- The goal is to distinguish isomers or regioselectivity of homologs — NPOmix, a correlation-based approach, lacks that resolution, so Jaccard co-occurrence cannot certify isomer identity.
- The link will be reported as a confirmed metabolite structure without further work — unvalidated links still require proper elucidation to confirm the metabolite's planar (2D) and absolute (3D) chemical structure.

## Inputs

- NPOmix KNN output: voted GCF labels per deduplicated query MS/MS spectrum (e.g. the m/z 780 siderophore spectrum)
- Expected GCF labels (MIBiG/PoDP-validated links or antiSMASH-annotated BGCs, e.g. siderophore BGC of Achromobacter xylosoxidans NH44784-1996)
- BiG-SCAPE similarity matrix (similarity = 1 − raw distance, 0.7 cutoff) underlying the trained KNN classifier

## Outputs

- Per-spectrum Jaccard co-occurrence score between predicted and expected GCF labels
- Validated link table (e.g. siderophore_npomix_links.tsv) with spectrum-to-GCF assignments and co-occurrence values
- Precision metrics (top-1 and top-10) computed under the Jaccard co-occurrence threshold

## How to apply

After NPOmix projects each deduplicated MS/MS fingerprint into the KNN space (k = 3 trained on BGC fingerprints built as 1 − BiG-SCAPE raw distance, scores below the 0.7 cutoff treated as absent), collect the voted GCF labels per spectrum and compare them against expected labels. Compute the Jaccard co-occurrence between predicted and expected GCF label sets; a link is accepted when the Jaccard value is high (100% for the validated m/z 780 siderophore link to the Achromobacter xylosoxidans NH44784-1996 siderophore GCF). Benchmark performance the same way the article did: top-1 precision = 81.8% and top-10 precision = 92.9%, using similarity and biosynthetic class features with randomness equal to 0 under the Jaccard co-occurrence threshold. Write accepted links to a table such as siderophore_npomix_links.tsv.

## Related tools

- **NPOmix** (KNN classifier whose voted GCF labels are compared to expected labels via Jaccard co-occurrence to validate links) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarities (1 − raw distance, 0.7 cutoff) that define the GCF label space against which predictions are validated) — https://bigscape-corason.secondarymetabolites.org
- **GNPS** (Provides modified cosine pairwise scoring used to build the MS/MS similarity fingerprints whose GCF assignments are validated) — https://gnps.ucsd.edu
- **antiSMASH** (Labels candidate BGCs (e.g. as siderophores) providing the expected labels for the Jaccard co-occurrence check) — https://antismash.secondarymetabolites.org/#!/start

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git && wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1
```

## Evaluation signals

- The m/z 780 spectrum is assigned to the siderophore GCF containing the Achromobacter xylosoxidans NH44784-1996 BGC with 100% Jaccard co-occurrence between predicted and expected GCF labels.
- Reproduced benchmark precision under the Jaccard co-occurrence threshold matches top-1 = 81.8% and top-10 = 92.9% (randomness = 0, similarity + biosynthetic class features).
- KNN settings match the article: k = 3, similarity-only (or similarity + biosynthetic class) features, BGC similarity = 1 − BiG-SCAPE raw distance with scores below 0.7 treated as absent.
- The link table (e.g. siderophore_npomix_links.tsv) contains one row per deduplicated MassQL-filtered query spectrum with its voted GCF labels and co-occurrence score.
- Predicted links lacking experimental validation are flagged for downstream structure elucidation rather than reported as confirmed.

## Limitations

- The Jaccard-validated precision may not be reproducible for every dataset, especially datasets dissimilar to the current PoDP-derived training set.
- New links validated only by co-occurrence are not experimentally confirmed; they require proper structure elucidation (planar 2D and absolute 3D).
- Metagenome quality can hinder BGC assembly, producing fragmented BGCs and fewer GCF labels, which degrades co-occurrence-based validation in microbiomes.

## Evidence

- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] similarity score of 1.0, a score of 0.8 would represent that a homologous BGC is present: "Similarity scores range from 0.0 to 1.0; for instance, identical BGCs have a perfect similarity score of 1.0, a score of 0.8 would represent that a homologous BGC is present in the genome, and the"
- [abstract] we demonstrate this approach by automatically linking 18 previously known mass spectra: [evidence span withheld — anchor status: not_found]
- [readme] obtain the GCF labels for the k-nearest neighbors to the query MS/MS fingerprint (e.g. for three most similar BGC neighbors, k = 3): "the algorithm can obtain the GCF labels for the k-nearest neighbors to the query MS/MS fingerprint (e.g. for three most similar BGC neighbors, k = 3)"
