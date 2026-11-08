---
name: siderophore-msms-spectra-dereplication-to-mgf
description: Use when a MassQL iron-bound siderophore query has returned redundant
  MS/MS hits (e.g. 380 putative siderophore spectra from thousands of PoDP LC-MS/MS
  profiles) and a deduplicated MGF is needed before NPOmix fingerprint computation.
when_to_use_negative:
- The spectra are already a nonredundant set (e.g. GNPS library reference spectra
  only) — in that case the 'not dereplication' NPOmix notebook variant applies instead
- The dataset lacks paired genome–metabolome samples or the query species has no BGC
  similar to the NPOmix training set (e.g. a fungal species with no similar BGCs produces
  no output)
- The goal is distinguishing isomers or regioselectivity of homologs — NPOmix, a correlation-based
  approach, does not have that resolution
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: NPOmix
  role: Provides the dereplication notebook logic that collapses MassQL-filtered siderophore
    spectra into a nonredundant MGF of query spectra
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: MassQL
  role: Iron-bound siderophore query language used upstream to extract the putative
    siderophore spectra from the pooled LC-MS/MS dataset
- name: GNPS
  role: Computes the pairwise modified cosine scores between the dereplicated query
    spectra and each paired experimental sample's spectra to build the MS/MS fingerprints
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:8962c6b86f5b256573b0f7a9ce9319ce8dacc8812fc0da19ab6ed46792137c5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - MassQL
  - NPOmix
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# siderophore-msms-spectra-dereplication-to-mgf

## Summary

Dereplicates MassQL-filtered iron-bound siderophore MS/MS spectra from pooled paired genome–metabolome LC-MS/MS data into a nonredundant MGF file of query spectra, which is the prerequisite input for NPOmix KNN linking of MS/MS fingerprints to siderophore GCFs.

## When to use

Use when a MassQL iron-bound siderophore query has returned redundant MS/MS hits (e.g. 380 putative siderophore spectra from thousands of PoDP LC-MS/MS profiles) and a deduplicated MGF is needed before NPOmix fingerprint computation.

## When NOT to use

- The spectra are already a nonredundant set (e.g. GNPS library reference spectra only) — in that case the 'not dereplication' NPOmix notebook variant applies instead
- The dataset lacks paired genome–metabolome samples or the query species has no BGC similar to the NPOmix training set (e.g. a fungal species with no similar BGCs produces no output)
- The goal is distinguishing isomers or regioselectivity of homologs — NPOmix, a correlation-based approach, does not have that resolution

## Inputs

- MassQL-filtered putative siderophore MS/MS spectra (380 spectra, Dataset S1 sheet 7) from pooled PoDP LC-MS/MS profiles
- GNPS job output from cloned task b915d1f78c364bf4be9900047760d95c containing training-set metabolomes
- NPOmix dereplication Jupyter notebook from github.com/tiagolbiotech/NPOmix_python

## Outputs

- Nonredundant MGF file of query siderophore MS/MS spectra
- Downstream MS/MS similarity fingerprint matrix (query spectra × 1,040 paired-sample columns of maximum modified cosine scores)

## How to apply

After the MassQL iron-bound siderophore query extracts the putative siderophore spectra from the pooled PoDP LC-MS/MS dataset (matching Dataset S1, sheet 7, ~380 spectra), run the NPOmix dereplication notebook logic to collapse redundant spectra into a nonredundant MGF file of query MS/MS spectra. Adjust the notebook paths to point at the MassQL output, the GNPS job output (e.g. cloned from GNPS task b915d1f78c364bf4be9900047760d95c), and the antiSMASH/BiG-SCAPE files. The dereplicated MGF is then used to compute, via GNPS pairwise modified cosine scoring, the maximum modified cosine similarity of each query spectrum to every spectrum in each of the 1,040 paired experimental samples, forming the MS/MS fingerprint matrix. Rationale: redundancy across samples would overweight duplicated spectra in the KNN feature space, so deduplication precedes fingerprint construction; NPOmix ships both 'dereplication' and 'not dereplication' notebook variants, and the dereplication variant is the one used in the combined MassQL + NPOmix siderophore workflow.

## Related tools

- **NPOmix** (Provides the dereplication notebook logic that collapses MassQL-filtered siderophore spectra into a nonredundant MGF of query spectra) — https://github.com/tiagolbiotech/NPOmix_python
- **MassQL** (Iron-bound siderophore query language used upstream to extract the putative siderophore spectra from the pooled LC-MS/MS dataset)
- **GNPS** (Computes the pairwise modified cosine scores between the dereplicated query spectra and each paired experimental sample's spectra to build the MS/MS fingerprints)

## Evaluation signals

- Dereplicated MGF covers the 380 putative siderophore spectra corresponding to Dataset S1 (sheet 7) as nonredundant query entries
- The m/z 780 query spectrum is retained in the MGF and later projects into the KNN space near the siderophore GCF containing the Achromobacter xylosoxidans NH44784-1996 BGC with 100% Jaccard co-occurrence
- The downstream fingerprint matrix has exactly 1,040 sample columns matching the 1,040 sets of paired experimental MS/MS spectra
- Modified cosine similarity scores fall in the 0.0–1.0 range used for the fingerprint matrix (analogous to BiG-SCAPE 1 − raw distance with a 0.7 cutoff)

## Limitations

- Precision scores (top-1 = 81.8%, top-10 = 92.9%) may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set
- NPOmix cannot distinguish isomers or regioselectivity of homologs; new links require proper structure elucidation to confirm the metabolite's planar (2D) and absolute (3D) chemistry
- Metagenome quality can hinder BGC assembly in microbiomes, producing fewer and fragmented BGCs on the genome-side of the linkage

## Evidence

- [discussion] combining NPOmix with MassQL can prioritize the discovery of putatively new siderophores: "combining NPOmix with MassQL can prioritize the discovery of putatively new siderophores"
- [readme] Run the dereplication or not dereplication notebooks. Make sure to adjust the paths for the antiSMASH, GNPS and BiG-SCAPE files.: "Run the dereplication or not dereplication notebooks. Make sure to adjust the paths for the antiSMASH, GNPS and BiG-SCAPE files."
- [readme] we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
- [abstract] We illustrated a computational example of how to use our Natural Products Mixed Omics (NPOmix) tool for siderophore mining that can be reproduced by the users.: "We illustrated a computational example of how to use our Natural Products Mixed Omics (NPOmix) tool for siderophore mining that can be reproduced by the users."
