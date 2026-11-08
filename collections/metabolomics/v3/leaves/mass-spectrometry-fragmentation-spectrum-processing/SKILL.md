---
name: mass-spectrometry-fragmentation-spectrum-processing
description: Use when you have paired genome + LC-MS/MS samples and need to turn raw
  fragmentation spectra (GNPS job output or cryptic spectra) into per-sample MS/MS
  fingerprints for BGC linking or benchmarking.
when_to_use_negative:
- The query organism has no BGCs similar to the NPOmix training set (e.g., a fungal
  species with no BGC similar to any PoDP-derived training BGC) — output would contain
  no answer
- The goal is distinguishing isomers or regioselectivity of homologs — the correlation-based
  approach lacks that resolution
- Only raw spectra without any paired genome/metagenome data exist, since linking
  requires the genomic fingerprint side
edam_topics:
- http://edamontology.org/topic_3364
tools:
- name: GNPS
  role: Computes pairwise modified cosine similarity scores between MS/MS spectra
    to build MS/MS fingerprints; hosts the metabolome job task=b915d1f78c364bf4be9900047760d95c
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: NPOmix
  role: k-NN classifier that consumes the MS/MS fingerprint testing matrix plus BGC
    fingerprint training matrix to assign GCF labels to spectra
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: tandem mass spectrometry
  role: Acquisition technique producing the untargeted LC-MS/MS fragmentation spectra
    that constitute the query and reference spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:87edb10d3e593e255d2433b892f547b34b2f180a52b4c6ac38cf2ee9e94e31a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - NPOmix
  - NRPminer
  - antiSMASH
  - tandem mass spectrometry
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-spectrometry-fragmentation-spectrum-processing

## Summary

Processes untargeted LC-MS/MS fragmentation spectra into MS/MS similarity fingerprints for BGC–metabolite linking: GNPS-computed pairwise modified cosine scores are aggregated per sample into a testing matrix that feeds the k-nearest neighbor classifier (NPOmix) and enables benchmarking linking precision/recall against validated links.

## When to use

Use when you have paired genome + LC-MS/MS samples and need to turn raw fragmentation spectra (GNPS job output or cryptic spectra) into per-sample MS/MS fingerprints for BGC linking or benchmarking.

## When NOT to use

- The query organism has no BGCs similar to the NPOmix training set (e.g., a fungal species with no BGC similar to any PoDP-derived training BGC) — output would contain no answer
- The goal is distinguishing isomers or regioselectivity of homologs — the correlation-based approach lacks that resolution
- Only raw spectra without any paired genome/metagenome data exist, since linking requires the genomic fingerprint side

## Inputs

- Paired genome + LC-MS/MS samples from the Paired Omics Data Platform (PoDP)
- GNPS job output with metabolome MS/MS spectra (e.g., task=b915d1f78c364bf4be9900047760d95c)
- antiSMASH BGC .gbk files (training set from zenodo.org/record/6637083, antismash_only_gbk.zip)
- BiG-SCAPE pairwise similarity outputs for BGC fingerprints

## Outputs

- MS/MS fingerprint testing matrix (rows = query spectra, columns = per-sample maximum modified cosine scores)
- KNN-predicted gene cluster family (GCF) labels per query MS/MS spectrum
- Top-1 precision and recall metrics per linking tool, comparable to the NPOmix benchmark (92.3% precision, 100% recall)

## How to apply

Retrieve the metabolome side of each paired sample — for the NPOmix validation this is the GNPS job task=b915d1f78c364bf4be9900047760d95c covering the samples holding 22 experimentally validated BGC–metabolite links from the Paired Omics Data Platform. Use GNPS to compute pairwise modified cosine scores between a query MS/MS spectrum (a GNPS library reference spectrum or a cryptic spectrum from a new sample) and every MS/MS spectrum in each experimental sample, taking the maximum similarity per sample to form the fingerprint vector. Note that full classical GNPS molecular networking is NOT used — only the modified cosine similarity functions. Scores range 0.0 to 1.0 (identical spectra = 1.0), and the analogue BGC-side cutoff is 0.7, below which a cluster is treated as absent. Stack the per-sample fingerprint rows into a testing matrix whose columns are the experimental MS/MS spectra per sample, feed it together with the labeled BGC training matrix into the KNN classifier (e.g., k = 3), then score top-1 predicted links against the validated links to compute top-1 precision and recall.

## Related tools

- **GNPS** (Computes pairwise modified cosine similarity scores between MS/MS spectra to build MS/MS fingerprints; hosts the metabolome job task=b915d1f78c364bf4be9900047760d95c) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **NPOmix** (k-NN classifier that consumes the MS/MS fingerprint testing matrix plus BGC fingerprint training matrix to assign GCF labels to spectra) — https://github.com/tiagolbiotech/NPOmix_python
- **tandem mass spectrometry** (Acquisition technique producing the untargeted LC-MS/MS fragmentation spectra that constitute the query and reference spectra)

## Evaluation signals

- Modified cosine scores are bounded in [0.0, 1.0], and fingerprints below the similarity cutoff of 0.7 are treated as absent (score 0.0)
- Testing matrix column count equals the number of paired experimental MS/MS sample sets (e.g., 1,040 in the reference release); training matrix columns match
- Predicted top-1 links scored against the 22 validated PoDP links reproduce NPOmix top-1 precision = 92.3% and recall = 100% using similarity, substructure, and biosynthetic class features with threshold
- Full GNPS molecular networking output is NOT present — only modified cosine pairwise scores are used
- Notebooks run without path errors pointing at the antiSMASH, GNPS, and BiG-SCAPE input folders

## Limitations

- NPOmix does not distinguish isomers or regioselectivity of homologs (correlation-based resolution limit)
- Precision may not be reproducible for datasets dissimilar to the PoDP-derived training set
- Fungal or other species with no BGC similar to the training set yield no answer; low-quality metagenomes yield fragmented/fewer BGCs that degrade linking
- No RiPP-class validated metabolites exist in the current PoDP training data, and paired plant data are lacking

## Evidence

- [readme] In the case of MS/MS fingerprints (Fig. 1D), we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample.: "In the case of MS/MS fingerprints (Fig. 1D), we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum"
- [readme] Similarity scores range from 0.0 to 1.0; for instance, identical BGCs have a perfect similarity score of 1.0: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [abstract] Untargeted metabolomics using liquid chromatography coupled with tandem mass spectrometry is an efficient technique to access metabolites: "Untargeted metabolomics using liquid chromatography coupled with tandem mass spectrometry is an efficient technique to access metabolites"
- [results] NPOmix achieved the highest precision (92.3%, using similarity, substructure, and biosynthetic class features with threshold) together with 100% recall: [evidence span withheld — anchor status: not_found]
- [methods] Retrieve the paired validation samples (genome + LC-MS/MS) for the 9 samples containing the 22 experimentally validated BGC–metabolite links: [evidence span withheld — anchor status: not_found]
