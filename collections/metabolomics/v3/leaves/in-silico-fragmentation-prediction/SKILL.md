---
name: in-silico-fragmentation-prediction
description: Use when a cryptic MS/MS spectrum has no GNPS library match (absent from
  the MS/MS database) and you need candidate structures plus predicted fragment masses
  to feed into NPOmix BGC–spectrum linking or to expand a mass spectral reference
  database.
when_to_use_negative:
- When the cryptic spectrum already has a confident GNPS library match — in-silico
  prediction is only needed for spectra absent from the MS/MS database.
- When the goal is to confirm a metabolite's planar (2D) and absolute (3D) chemical
  structure — the article states new links 'require proper elucidation'; in-silico
  predictions alone cannot confirm structure.
- When the query BGC has no similar BGC in the NPOmix training dataset (e.g. a fungal
  species with no similar BGCs) — the article notes the output would contain no answer.
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MZmine
  role: Preprocesses LC-MS/MS spectra to produce deconvoluted, aligned MS/MS peak
    lists used as input to Dereplicator+
- name: Dereplicator+
  role: Performs in-silico fragmentation on each preprocessed cryptic MS/MS spectrum
    to produce predicted candidate structures and their theoretical fragment spectra
- name: GNPS
  role: Provides the molecular networking job cloned to add candidate spectra and
    compute pairwise modified cosine scores for the testing matrix
- name: NPOmix
  role: Consumes the testing matrix built from candidate MS/MS fingerprints to assign
    gene cluster family (GCF) labels via KNN
  repo: https://github.com/tiagolbiotech/NPOmix_python
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:5a71edab6ade21a5de444e0f2485d974e8102be655db782c8b800b3878d8502e
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
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# in-silico-fragmentation-prediction

## Summary

Predict theoretical fragment spectra for candidate metabolite structures using an in-silico fragmentation tool (Dereplicator+), then match predicted fragment masses against observed cryptic MS/MS spectra to generate 'level bronze' metabolite links for downstream BGC–spectrum linking.

## When to use

Use when a cryptic MS/MS spectrum has no GNPS library match (absent from the MS/MS database) and you need candidate structures plus predicted fragment masses to feed into NPOmix BGC–spectrum linking or to expand a mass spectral reference database.

## When NOT to use

- When the cryptic spectrum already has a confident GNPS library match — in-silico prediction is only needed for spectra absent from the MS/MS database.
- When the goal is to confirm a metabolite's planar (2D) and absolute (3D) chemical structure — the article states new links 'require proper elucidation'; in-silico predictions alone cannot confirm structure.
- When the query BGC has no similar BGC in the NPOmix training dataset (e.g. a fungal species with no similar BGCs) — the article notes the output would contain no answer.

## Inputs

- Preprocessed, deconvoluted LC-MS/MS spectra from cryptic metabolite features (e.g. MZmine peak lists)
- Raw LC-MS/MS spectra requiring MZmine preprocessing into MS/MS peak lists
- GNPS molecular networking job (e.g. task=b915d1f78c364bf4be9900047760d95c) to clone and extend with candidate spectra

## Outputs

- Predicted candidate structures per cryptic MS/MS spectrum from Dereplicator+
- Theoretical fragment spectra matched to observed cryptic spectra (delta m/z < 0.001 verification)
- New MS/MS library entries ('level bronze' metabolite links) ready for NPOmix BGC–spectrum classification

## How to apply

Preprocess LC-MS/MS data with MZmine to obtain deconvoluted, aligned MS/MS peak lists, then run Dereplicator+ in-silico fragmentation on each preprocessed cryptic spectrum to produce predicted candidate structures and their theoretical fragment spectra. Clone the GNPS molecular networking job (task=b915d1f78c364bf4be9900047760d95c), add the candidate spectra, and compute pairwise modified cosine scores between each candidate MS/MS fingerprint and every experimental sample to build the testing matrix. Verify predicted structures by comparing Dereplicator+ predicted fragment masses against the observed spectrum, requiring delta m/z < 0.001 for a fragment match. The rationale: spectra absent from GNPS cannot be dereplicated by library search alone, so combining MZmine, Dereplicator+, and NPOmix creates new links that expand the MS/MS database as 'level bronze' metabolites.

## Related tools

- **MZmine** (Preprocesses LC-MS/MS spectra to produce deconvoluted, aligned MS/MS peak lists used as input to Dereplicator+)
- **Dereplicator+** (Performs in-silico fragmentation on each preprocessed cryptic MS/MS spectrum to produce predicted candidate structures and their theoretical fragment spectra)
- **GNPS** (Provides the molecular networking job cloned to add candidate spectra and compute pairwise modified cosine scores for the testing matrix)
- **NPOmix** (Consumes the testing matrix built from candidate MS/MS fingerprints to assign gene cluster family (GCF) labels via KNN) — https://github.com/tiagolbiotech/NPOmix_python

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git && wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1
```

## Evaluation signals

- Predicted fragment masses from Dereplicator+ must match observed cryptic spectrum fragments within delta m/z < 0.001
- The candidate spectrum should be absent from the GNPS library (no library match) — if it matches, in-silico prediction is unnecessary
- The testing matrix must contain pairwise modified cosine scores between each candidate MS/MS fingerprint and every experimental sample
- The predicted structure's m/z should match the literature m/z for the target metabolite (e.g. brasilicardin A)
- The pipeline output should enable NPOmix to assign the correct GCF (e.g. MIBiG BGC0000632) using only similarity-score features

## Limitations

- In-silico predictions create 'level bronze' metabolite links that still require proper elucidation to confirm the metabolite's planar (2D) and absolute (3D) chemical structure.
- The article notes putative orfamide predictions were not confirmed via structure elucidation or co-elution with standards.
- NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs.
- The precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set.

## Evidence

- [discussion] combine in-silico tools like Mzmine, Dereplicator+, and NPOmix to create new links that can expand MS/MS mass spectral reference database: "combine in-silico tools like Mzmine, Dereplicator+, and NPOmix to create new links that can expand MS/MS mass spectral reference database as "level bronze" metabolites"
- [discussion] spectra from cryptic metabolites (links without GNPS library matches and therefore absent from the MS/MS database: "spectra from cryptic metabolites (links without GNPS library matches and therefore absent from the MS/MS database"
- [discussion] new links (not experimentally validated links that are, for example, reported in the PoDP database) require proper elucidation: [evidence span withheld — anchor status: not_found]
- [readme] Run Dereplicator+ in-silico fragmentation on each preprocessed spectrum to produce predicted candidate structures and their theoretical fragment spectra.: [evidence span withheld — anchor status: not_found]
- [readme] require a similarity cutoff of 0.7 for the BiG-SCAPE BGC fingerprint training matrix: "similarity scores range from 0.0 to 1.0; for instance, identical BGCs have a perfect similarity score of 1.0, a score of 0.8 would represent that a homologous BGC is present in the genome, and the"
- [readme] the modified cosine score is computed between a pair of MS/MS spectra: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
