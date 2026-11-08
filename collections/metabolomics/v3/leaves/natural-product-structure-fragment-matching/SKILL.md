---
name: natural-product-structure-fragment-matching
description: Use when a cryptic MS/MS spectrum absent from the GNPS library must be
  connected to a candidate structure and its biosynthetic gene cluster via predicted-fragment
  fingerprints.
when_to_use_negative:
- The query organism's BGCs share no similarity (below the 0.7 BiG-SCAPE cutoff) with
  any BGC in the PoDP-derived NPOmix training set — the classifier returns no answer
- The MS/MS spectrum already has a confident GNPS library match and dereplication
  by in-silico prediction would be redundant
- Goal is resolving isomers or regioselectivity of homologs — correlation-based NPOmix
  cannot distinguish them
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3314
tools:
- name: MZmine
  role: Preprocesses LC-MS/MS spectra of cryptic metabolite features into deconvoluted,
    aligned MS/MS peak lists for in-silico prediction and fingerprint construction
- name: Dereplicator+
  role: Generates in-silico predicted candidate structures (including brasilicardin
    A) and theoretical fragment spectra used to build the testing matrix fingerprints
- name: NPOmix
  role: KNN classifier that assigns gene cluster family labels to each candidate MS/MS
    fingerprint using similarity BGC fingerprints trained from BiG-SCAPE scores
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (cutoff 0.7) that form the training-matrix
    fingerprints and assign BGCs to gene cluster families
- name: GNPS
  role: Source of the cloned molecular networking job (task=b915d1f78c364bf4be9900047760d95c)
    used to compute pairwise modified cosine scores between candidate and experimental
    MS/MS spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:b55cabf4e0726f0268bafa291b442954af2b4f1a3f8d130e425df35e0a47d5f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - Dereplicator+
  - GNPS
  - MZmine
  - NPOmix
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# natural-product-structure-fragment-matching

## Summary

Links in-silico–predicted natural product structures to observed cryptic MS/MS spectra by comparing predicted fragment fingerprints against experimental fragmentation, as in the NPOmix brasilicardin A workflow where MZmine-processed spectra were run through Dereplicator+ and connected to MIBiG BGC0000632.

## When to use

Use when a cryptic MS/MS spectrum absent from the GNPS library must be connected to a candidate structure and its biosynthetic gene cluster via predicted-fragment fingerprints.

## When NOT to use

- The query organism's BGCs share no similarity (below the 0.7 BiG-SCAPE cutoff) with any BGC in the PoDP-derived NPOmix training set — the classifier returns no answer
- The MS/MS spectrum already has a confident GNPS library match and dereplication by in-silico prediction would be redundant
- Goal is resolving isomers or regioselectivity of homologs — correlation-based NPOmix cannot distinguish them

## Inputs

- MZmine-preprocessed, deconvoluted MS/MS peak lists from paired genome–metabolome samples
- Dereplicator+ in-silico candidate structures with predicted fragment spectra
- antiSMASH BGC GenBank files (e.g. antismash_only_gbk.zip from Zenodo record 6637083)
- GNPS job output containing pairwise modified cosine scores (task=b915d1f78c364bf4be9900047760d95c)
- BiG-SCAPE pairwise similarity outputs (e.g. 2023-11-11_11-11-11_hybrids_glocal/) concatenated across BGC classes

## Outputs

- BGC fingerprint training matrix (rows = maximum BiG-SCAPE similarity scores, columns = 1,040 genomes)
- MS/MS testing matrix linking candidate fingerprints to experimental samples
- GCF labels for each cryptic spectrum from the NPOmix KNN classifier
- Fragment-match table confirming predicted vs. observed fragments (delta m/z < 0.001), e.g. npomix_brasilicardin_link_results.csv

## How to apply

Preprocess LC-MS/MS data with MZmine to obtain deconvoluted, aligned peak lists, then run Dereplicator+ on each spectrum to generate predicted candidate structures and their theoretical fragment spectra. Clone the GNPS job (task=b915d1f78c364bf4be9900047760d95c), add the candidate spectra, and compute pairwise modified cosine scores between each candidate MS/MS fingerprint and every experimental sample to build the testing matrix. Train the NPOmix k-nearest neighbor classifier (k = 3) on BiG-SCAPE similarity BGC fingerprints (similarity cutoff 0.7, rows = maximum similarity of the query BGC to BGCs in each of 1,040 paired genomes) and feed the testing matrix to obtain gene cluster family labels. Verify the top-ranked link by comparing Dereplicator+ predicted fragment masses against the observed spectrum requiring delta m/z < 0.001, as done for brasilicardin A against MIBiG BGC0000632 (Fig. S5).

## Related tools

- **MZmine** (Preprocesses LC-MS/MS spectra of cryptic metabolite features into deconvoluted, aligned MS/MS peak lists for in-silico prediction and fingerprint construction)
- **Dereplicator+** (Generates in-silico predicted candidate structures (including brasilicardin A) and theoretical fragment spectra used to build the testing matrix fingerprints)
- **NPOmix** (KNN classifier that assigns gene cluster family labels to each candidate MS/MS fingerprint using similarity BGC fingerprints trained from BiG-SCAPE scores) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarity scores (cutoff 0.7) that form the training-matrix fingerprints and assign BGCs to gene cluster families)
- **GNPS** (Source of the cloned molecular networking job (task=b915d1f78c364bf4be9900047760d95c) used to compute pairwise modified cosine scores between candidate and experimental MS/MS spectra)

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git && wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1
```

## Evaluation signals

- Predicted fragments from Dereplicator+ match the observed brasilicardin A spectrum within delta m/z < 0.001 (reproducing Fig. S5)
- Top-ranked GCF for the linked spectrum corresponds to the expected MIBiG BGC (BGC0000632 from Nocardia terpenica IFM 0406)
- Dataset-level precision: top-1 = 81.8% and top-10 = 92.9% using similarity + biosynthetic class features with the Jaccard co-occurrence threshold
- Training matrix rows should be maximum BiG-SCAPE similarity scores (0.0–1.0; identical BGCs = 1.0, absent = 0.0 or below the 0.7 cutoff) with one extra column labeling each BGC fingerprint with its GCF
- BGC fingerprints below the 0.7 similarity cutoff are treated as absent (score 0.0), and dereplication notebook paths must point to the antiSMASH, GNPS, and BiG-SCAPE files

## Limitations

- Precision scores may not be reproducible for datasets dissimilar to the current PoDP training set
- Fungal species with no BGC similar to any training-set BGC produce no output
- NPOmix cannot distinguish isomers or regioselectivity of homologs
- Metagenome quality can hinder BGC assembly, producing fragmented BGCs and fewer clusters than expected
- No RiPP class validated metabolites exist in the PoDP training data, and paired plant data are largely absent
- New links require proper structure elucidation to confirm planar (2D) and absolute (3D) chemistry

## Evidence

- [results] Screens 300 cryptic metabolite features with Dereplicator+ whose predictions include brasilicardin A: [evidence span withheld — anchor status: not_found]
- [methods] GNPS job used to compute pairwise modified cosine scores: [evidence span withheld — anchor status: not_found]
- [methods] BiG-SCAPE similarity cutoff for BGC fingerprints: [evidence span withheld — anchor status: not_found]
- [methods] KNN classifier parameter: [evidence span withheld — anchor status: not_found]
- [results] Fragment-match validation threshold: [evidence span withheld — anchor status: not_found]
- [results] Dataset-level precision metrics: [evidence span withheld — anchor status: not_found]
- [discussion] Known limitation: precision reproducibility: "the precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set"
- [discussion] Known limitation: no-answer case for dissimilar BGCs: "by submitting a fungal species that has no BGC similar to any BGC in the NPOmix training dataset (obtained from the PoDP database), the output would contain no answer"
- [discussion] Known limitation: isomer discrimination: "NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs"
