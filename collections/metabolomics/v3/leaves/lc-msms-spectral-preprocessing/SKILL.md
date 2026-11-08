---
name: lc-msms-spectral-preprocessing
description: Use when raw LC-MS/MS spectra from paired genome–metabolome samples (e.g.
  PoDP bacterial/fungal strains or cryptic features from MZmine) must be deconvoluted,
  aligned, and converted into MS/MS fingerprints for NPOmix KNN linking of spectra
  to biosynthetic gene clusters.
when_to_use_negative:
- When the input is already an aligned MS/MS peak list or a finished GNPS job output
  — skip straight to the modified-cosine fingerprint computation
- When the goal is genome-only BGC prediction without any spectral data (e.g. pure
  antiSMASH/BiG-SCAPE similarity analysis); spectral preprocessing is irrelevant
- When spectra belong to non-bacterial organisms lacking comparable paired datasets
  in the NPOmix training set — the article notes a current lack of paired data for
  plants and that fungal species with no similar BGCs produce no output
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MZmine
  role: Preprocesses raw LC-MS/MS files into deconvoluted, aligned MS/MS peak lists
    that serve as input for GNPS similarity scoring and downstream NPOmix fingerprints.
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: GNPS
  role: Hosts the molecular-networking job used to compute pairwise modified cosine
    similarity scores between query MS/MS spectra and experimental samples, producing
    the MS/MS fingerprint testing matrix.
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: NPOmix
  role: Consumes the MS/MS fingerprints generated from preprocessed spectra, pairing
    them with BiG-SCAPE BGC fingerprints to train the k-nearest-neighbor classifier
    linking spectra to gene cluster families.
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: Paired Omics Data Platform (PoDP)
  role: Provides paired genome–metabolome training datasets whose MS/MS spectra are
    preprocessed and fingerprinted to train NPOmix.
  repo: https://pairedomicsdata.bioinformatics.nl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:ed2362e120b6d868921671835c72bc45be6d1b5cdeb477b49101f048f3c2bf0a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - Dereplicator+
  - GNPS
  - MZmine
  - NPOmix
  - Paired Omics Data Platform (PoDP)
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 30
---

# lc-msms-spectral-preprocessing

## Summary

Preprocess untargeted LC-MS/MS metabolome files (MZmine-style deconvolution and alignment, then GNPS molecular-networking jobs) to produce aligned MS/MS peak lists that feed NPOmix's modified-cosine similarity fingerprints.

## When to use

Use when raw LC-MS/MS spectra from paired genome–metabolome samples (e.g. PoDP bacterial/fungal strains or cryptic features from MZmine) must be deconvoluted, aligned, and converted into MS/MS fingerprints for NPOmix KNN linking of spectra to biosynthetic gene clusters.

## When NOT to use

- When the input is already an aligned MS/MS peak list or a finished GNPS job output — skip straight to the modified-cosine fingerprint computation
- When the goal is genome-only BGC prediction without any spectral data (e.g. pure antiSMASH/BiG-SCAPE similarity analysis); spectral preprocessing is irrelevant
- When spectra belong to non-bacterial organisms lacking comparable paired datasets in the NPOmix training set — the article notes a current lack of paired data for plants and that fungal species with no similar BGCs produce no output

## Inputs

- Raw LC-MS/MS data files from paired genome–metabolome samples (untargeted metabolomics, e.g. from PoDP bacterial or fungal strains)
- Cryptic MS/MS spectra from a new sample (spectra absent from GNPS library matches)
- GNPS reference spectra for training-set MS/MS fingerprints

## Outputs

- MZmine-deconvoluted and aligned MS/MS peak lists per sample
- GNPS molecular-networking job outputs containing pairwise modified cosine similarity scores
- MS/MS fingerprint testing matrix (rows = query MS/MS spectra, columns = experimental samples) ready for NPOmix KNN classification

## How to apply

Run MZmine preprocessing on the LC-MS/MS files to obtain deconvoluted, aligned MS/MS peak lists per sample (merging multiple LC-MS/MS files for the same genome into one set of experimental spectra). Clone the GNPS molecular-networking job (task=b915d1f78c364bf4be9900047760d95c) and add the sample spectra to the G3 group, downloading the resulting GNPS output. NPOmix consumes these processed spectra to compute pairwise modified cosine scores between each query MS/MS spectrum and every experimental sample, building the MS/MS fingerprint testing matrix where columns are the paired experimental spectra per sample. Note that full classical GNPS molecular networking is not required — only the modified cosine similarity scoring between spectral pairs is needed. The resulting fingerprints (e.g. 15 reference spectra from GNPS for the first release, thousands of rows for larger sets) are then paired with BiG-SCAPE similarity-score BGC fingerprints (cutoff 0.7) to train the k-nearest-neighbor classifier (k=3) that assigns gene-cluster-family labels to cryptic spectra.

## Related tools

- **MZmine** (Preprocesses raw LC-MS/MS files into deconvoluted, aligned MS/MS peak lists that serve as input for GNPS similarity scoring and downstream NPOmix fingerprints.) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **GNPS** (Hosts the molecular-networking job used to compute pairwise modified cosine similarity scores between query MS/MS spectra and experimental samples, producing the MS/MS fingerprint testing matrix.) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **NPOmix** (Consumes the MS/MS fingerprints generated from preprocessed spectra, pairing them with BiG-SCAPE BGC fingerprints to train the k-nearest-neighbor classifier linking spectra to gene cluster families.) — https://github.com/tiagolbiotech/NPOmix_python
- **Paired Omics Data Platform (PoDP)** (Provides paired genome–metabolome training datasets whose MS/MS spectra are preprocessed and fingerprinted to train NPOmix.) — https://pairedomicsdata.bioinformatics.nl

## Examples

```
wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1 && unzip antismash_bgcs.zip && sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/
```

## Evaluation signals

- MZmine output must contain deconvoluted MS/MS peak lists aligned across samples (one merged set of experimental spectra per genome when multiple LC-MS/MS files exist for the same strain)
- GNPS job must return pairwise modified cosine scores between each query spectrum and every experimental sample, forming a testing matrix whose columns equal the number of paired experimental spectra (e.g. 1,040 in the first release)
- MS/MS fingerprint values must lie in the range 0.0–1.0, with identical spectra scoring 1.0 and scores below the 0.7 similarity cutoff representing absence in a sample (analogous to the BiG-SCAPE BGC fingerprint convention)
- Downstream NPOmix KNN classification should achieve reported dataset-level performance: top-1 precision 81.8% and top-10 precision 92.9% (with the Jaccard co-occurrence threshold on similarity plus biosynthetic-class features)

## Limitations

- Metagenome quality can hinder BGC assembly, leading to fewer and fragmented BGCs, which weakens the genome side of the paired fingerprint
- The precision score may not be reproducible for every dataset, especially if the dataset differs substantially from the current training set
- NPOmix cannot distinguish isomers or regioselectivity of homologs because it is a correlation-based approach
- Fungal species with no BGCs similar to those in the NPOmix training dataset yield no output

## Evidence

- [methods] Clone the GNPS job with the metabolomes for the strains in the training set and add your files: "Clone the GNPS job with the metabolomes for the strains in the training set and add your files"
- [readme] By applying BiG-SCAPE, each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison: [evidence span withheld — anchor status: not_found]
- [readme] we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample: "In the case of MS/MS fingerprints (Fig. 1D), we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS"
- [readme] we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score: "Of particular note, we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score between a pair of"
- [discussion] combine in-silico tools like Mzmine, Dereplicator+, and NPOmix to create new links that can expand MS/MS mass spectral reference database: "combine in-silico tools like Mzmine, Dereplicator+, and NPOmix to create new links that can expand MS/MS mass spectral reference database as "level bronze" metabolites"
