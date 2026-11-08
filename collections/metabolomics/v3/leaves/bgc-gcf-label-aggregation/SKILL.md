---
name: bgc-gcf-label-aggregation
description: Use when paired genome–MS/MS samples need candidate GCF labels per MS/MS
  spectrum aggregated from kNN neighbor votes, and candidate links must be confidence-filtered
  against PoDP-validated gene–metabolite links.
when_to_use_negative:
- Query genomes (e.g. fungal species) have no BGC similar to any BGC in the NPOmix
  training set — the output contains no answer since the method requires similarity
  scores
- The goal is distinguishing isomers or regioselectivity of homologs — the correlation-based
  approach lacks that resolution
- Candidate links are novel (absent from PoDP/GNPS reference data) and require actual
  structure elucidation (planar 2D and absolute 3D chemistry) rather than computational
  label aggregation
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: NPOmix
  role: k-nearest neighbor classifier that aggregates GCF labels from MS/MS fingerprint
    neighbors and applies the Jaccard co-occurrence threshold
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: computes pairwise BGC similarity scores (1 minus raw distance) used to build
    BGC fingerprints and assign GCFs
  repo: https://bigscape-corason.secondarymetabolites.org
- name: GNPS
  role: computes modified cosine similarity scores between MS/MS spectra used to build
    MS/MS fingerprints
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: antiSMASH
  role: predicts the BGC .gbk files that feed the BiG-SCAPE similarity comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:532785a82362cf8cbde9ccf9478fd6d3187b14c0931ec039a122c467c433cfd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bgc-gcf-label-aggregation

## Summary

Aggregates k-nearest-neighbor GCF label votes over BGC and MS/MS similarity fingerprints to score candidate BGC–metabolite links, then applies a Jaccard co-occurrence threshold (0.7) as a confidence filter evaluated on PoDP-validated links.

## When to use

Use when paired genome–MS/MS samples need candidate GCF labels per MS/MS spectrum aggregated from kNN neighbor votes, and candidate links must be confidence-filtered against PoDP-validated gene–metabolite links.

## When NOT to use

- Query genomes (e.g. fungal species) have no BGC similar to any BGC in the NPOmix training set — the output contains no answer since the method requires similarity scores
- The goal is distinguishing isomers or regioselectivity of homologs — the correlation-based approach lacks that resolution
- Candidate links are novel (absent from PoDP/GNPS reference data) and require actual structure elucidation (planar 2D and absolute 3D chemistry) rather than computational label aggregation

## Inputs

- BiG-SCAPE pairwise similarity output between antiSMASH-predicted BGCs (concatenated per-class glocal files, --mibig enabled)
- GNPS modified-cosine similarity scores between query MS/MS spectra and per-sample experimental spectra
- GCF label assignments for training BGCs (BiG-SCAPE gene cluster families, MIBiG-anchored)
- Paired genome/metagenome–MS/MS sample metadata from the PoDP
- PoDP experimentally validated gene–metabolite links (ground truth)

## Outputs

- Per-query GCF label list with aggregated neighbor votes (k=3)
- Per-candidate-link Jaccard co-occurrence scores
- Filtered and unfiltered top-1/top-10 precision against PoDP validated links (jaccard_filter_metrics.csv per source task)
- Retained-link counts at evaluated co-occurrence cutoffs

## How to apply

Build a training matrix of BGC fingerprints (maximum BiG-SCAPE similarity score, 1 minus raw distance, per genome column; scores below the 0.7 similarity cutoff count as absent) labeled with GCF assignments, and a testing matrix of MS/MS fingerprints (maximum modified-cosine score per paired sample column, computed via GNPS). Feed the BGC matrix to the k-nearest neighbor classifier with similarity and biosynthetic class features, retrieving GCF labels for the k=3 most similar BGC neighbors; GCF labels may repeat when several neighbors share a gene family, which is expected kNN behavior. Aggregate the repeated GCF votes per query spectrum to rank candidate links, then compute a Jaccard co-occurrence index between the thresholded BGC fingerprint and the MS/MS fingerprint for each candidate link and retain links at the 0.7 co-occurrence threshold. Score retained vs. unfiltered links against PoDP-validated links as ground truth, targeting the reported precision of top-1 = 81.8% and top-10 = 92.9% (randomness equal to 0); the cutoff reduced 22 validated-candidate links to roughly 14 with 13 of 14 reference spectra correctly labeled.

## Related tools

- **NPOmix** (k-nearest neighbor classifier that aggregates GCF labels from MS/MS fingerprint neighbors and applies the Jaccard co-occurrence threshold) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (computes pairwise BGC similarity scores (1 minus raw distance) used to build BGC fingerprints and assign GCFs) — https://bigscape-corason.secondarymetabolites.org
- **GNPS** (computes modified cosine similarity scores between MS/MS spectra used to build MS/MS fingerprints) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **antiSMASH** (predicts the BGC .gbk files that feed the BiG-SCAPE similarity comparison)

## Examples

```
conda activate bigscape && bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- Applying the Jaccard co-occurrence threshold with similarity + biosynthetic class features reproduces reported precision top-1 = 81.8% and top-10 = 92.9% at randomness equal to 0
- 13 of 14 reference MS/MS spectra are correctly labeled at k=3 (92.9% precision) after the 0.7 cutoff
- Retained-link count drops from ~22 to ~14 candidate links when the 0.7 threshold is applied — record counts at every evaluated cutoff
- Repeated GCF labels within a k=3 neighbor list are counted, not deduplicated, before aggregation
- BGC fingerprint similarity scores below the 0.7 cutoff are treated as absent (0.0) in the training matrix

## Limitations

- Precision may not be reproducible for every dataset, especially if it is not similar to the current PoDP-derived training set
- No RiPP-class validated metabolites exist in the PoDP training data, so RiPP links cannot be validated
- Metagenome quality can hinder BGC assembly, yielding fewer and fragmented BGCs that weaken fingerprints
- The training matrix scale (5,421 BGCs across 1,040 genomes/metagenomes in release round 4) bounds the label space the kNN can vote over

## Evidence

- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] the algorithm can obtain the GCF labels for the k-nearest neighbors to the query MS/MS fingerprint (e.g., for three most similar BGC neighbors, k = 3): "the algorithm can obtain the GCF labels for the k-nearest neighbors to the query MS/MS fingerprint (e.g., for three most similar BGC neighbors, k = 3)"
- [readme] GCF labels can be present more than once in the returned list if two or more BGC nearest neighbors belong to the same gene family: "We note that GCF labels can be present more than once in the returned list if two or more BGC nearest neighbors belong to the same gene family."
- [readme] a score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome: "a score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome."
