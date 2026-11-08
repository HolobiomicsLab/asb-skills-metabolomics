---
name: msms-modified-cosine-spectral-matching
description: Use when you have paired genome–metabolome samples (PoDP-style, e.g.
  the 1,040-sample GNPS job task=b915d1f78c364bf4be9900047760d95c) and reference or
  cryptic MS/MS spectra that must be converted into similarity fingerprints for BGC–metabolite
  linking.
when_to_use_negative:
- Your goal is to distinguish isomers or regioselectivity of homologous metabolites
  — the article states NPOmix, a correlation-based approach, 'does not have the resolution
  to distinguish between isomers and regioselectivity of the various homologs'.
- The query organism has no BGC similar to any training BGC (e.g., a fungal species
  distant from the PoDP-derived training set) — the output would contain no answer,
  so spectral matching alone will not yield a link.
- You need validated links for plant metabolites or RiPP-class compounds — the article
  notes a lack of paired plant data and no RiPP-class validated metabolites in the
  PoDP training data.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Computes pairwise modified cosine similarity scores between query MS/MS spectra
    and experimental sample spectra (only the cosine-scoring function is used, not
    full molecular networking)
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: NPOmix
  role: Consumes the MS/MS fingerprint testing matrix and the BiG-SCAPE BGC training
    matrix in a scikit-learn k-nearest-neighbor classifier to assign GCF labels
  repo: https://github.com/tiagolbiotech/NPOmix_python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:e204b4f5a1c66f3bfb784117b650153a6f48bb7afa5e478db6c45ee251adc644
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - MIBiG database
  - NPOmix
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# msms-modified-cosine-spectral-matching

## Summary

Compute pairwise modified cosine similarity scores between a query MS/MS spectrum (e.g., a GNPS library reference such as CCMSLIB00004679300) and all experimental spectra in paired LC-MS/MS samples using GNPS, then condense the scores into an MS/MS fingerprint vector (maximum score per sample) for downstream GCF classification by NPOmix.

## When to use

Use when you have paired genome–metabolome samples (PoDP-style, e.g. the 1,040-sample GNPS job task=b915d1f78c364bf4be9900047760d95c) and reference or cryptic MS/MS spectra that must be converted into similarity fingerprints for BGC–metabolite linking.

## When NOT to use

- Your goal is to distinguish isomers or regioselectivity of homologous metabolites — the article states NPOmix, a correlation-based approach, 'does not have the resolution to distinguish between isomers and regioselectivity of the various homologs'.
- The query organism has no BGC similar to any training BGC (e.g., a fungal species distant from the PoDP-derived training set) — the output would contain no answer, so spectral matching alone will not yield a link.
- You need validated links for plant metabolites or RiPP-class compounds — the article notes a lack of paired plant data and no RiPP-class validated metabolites in the PoDP training data.

## Inputs

- GNPS molecular networking job output for 1,040 paired genome–MS/MS samples (task=b915d1f78c364bf4be9900047760d95c)
- Reference MS/MS spectra from the GNPS library (e.g., albicidin analog, orfamide C CCMSLIB00004679300; 22 spectra in the dereplication task)
- Cryptic MS/MS spectra from new paired samples (optional queries)
- Per-sample experimental peak lists / mgf files from untargeted LC-MS/MS

## Outputs

- MS/MS fingerprint vectors (rows = query spectra, columns = 1,040 paired samples, values = maximum modified cosine score per sample)
- Testing matrix for the NPOmix KNN classifier
- Predicted gene cluster family (GCF) labels per query spectrum, after applying the Jaccard co-occurrence threshold
- Top-1 and top-10 precision against validated PoDP/MIBiG links

## How to apply

Clone the GNPS job containing the 1,040 paired-sample metabolomes (task=b915d1f78c364bf4be9900047760d95c) and add your own files, then use GNPS to compute the pairwise modified cosine score between each query MS/MS spectrum (e.g., one of the 22 GNPS reference spectra used in the dereplication task) and every spectrum in each experimental sample. Note the article deliberately did not use full classical GNPS molecular networking — only the pairwise modified cosine similarity function. For each query spectrum, take the maximum modified cosine score against the spectra in each sample to form a fingerprint vector with one column per sample (1,040 columns for the training set). Binarize or treat scores below the 0.7 similarity cutoff as absent, mirroring the BGC fingerprint treatment, and feed the resulting testing matrix to the scikit-learn KNN classifier alongside the BiG-SCAPE-derived BGC training matrix.

## Related tools

- **GNPS** (Computes pairwise modified cosine similarity scores between query MS/MS spectra and experimental sample spectra (only the cosine-scoring function is used, not full molecular networking)) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **NPOmix** (Consumes the MS/MS fingerprint testing matrix and the BiG-SCAPE BGC training matrix in a scikit-learn k-nearest-neighbor classifier to assign GCF labels) — https://github.com/tiagolbiotech/NPOmix_python

## Evaluation signals

- On the validated GNPS reference dereplication task (k = 3, randomness = 0, similarity + biosynthetic class features, Jaccard co-occurrence threshold), top-10 precision should reach 0.929 (13/14 spectra) and top-1 precision 0.818.
- Each MS/MS fingerprint vector must contain one column per paired sample — 1,040 columns for the training GNPS job.
- The albicidin analog link and the CCMSLIB00004679300 putative orfamide C link should appear among the top-n predictions.
- Modified cosine scores below the 0.7 similarity cutoff should be treated as absent (binarized), consistent with BGC fingerprint construction.
- Only modified cosine scores (not full GNPS molecular networking results) should appear in the fingerprint matrix.

## Limitations

- NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs.
- The precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set.
- Putative links such as the orfamides were not confirmed via structure elucidation or co-elution with standards; new links require proper elucidation to confirm planar (2D) and absolute (3D) structure.
- Metagenome quality can hinder BGC assembly, leading to fewer and fragmented BGCs in microbiome samples.

## Evidence

- [readme] we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample.: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample."
- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score between a pair of MS/MS spectra.: "we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score between a pair of MS/MS spectra."
