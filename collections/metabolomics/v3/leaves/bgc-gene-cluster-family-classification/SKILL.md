---
name: bgc-gene-cluster-family-classification
description: Use when you have paired genome-MS/MS samples (e.g. PoDP genomes plus
  LC-MS/MS spectra), a query MS/MS spectrum (GNPS reference or cryptic), and want
  to link it to a gene cluster family with precision ~0.929 top-10.
when_to_use_negative:
- When the query genome or sample has no BGC similar to any BGC in the NPOmix training
  dataset (e.g. a fungal species with no homologs in the PoDP-derived training set)
  — the output will contain no answer
- When the input is a single unpaired genome or metabolome with no corresponding paired
  samples — the classifier requires paired genome-MS/MS co-occurrence to build fingerprints
- When distinguishing structural isomers or regioselectivity of homologous metabolites
  is required — NPOmix's correlation-based approach lacks this resolution
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: NPOmix
  role: k-nearest-neighbor classifier that assigns gene cluster family labels to query
    MS/MS fingerprints using similarity BGC and MS/MS fingerprints
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: computes pairwise similarity scores (1 minus raw distance) between BGCs to
    build the BGC fingerprint training matrix and assign GCFs (with --include_singletons
    --mibig and Pfam-A.hmm 35.0)
  repo: https://bigscape-corason.secondarymetabolites.org
- name: antiSMASH
  role: produces the training BGC .gbk files that are converted into BGC fingerprints;
    also used to annotate sample genomes included in the training matrix
  repo: https://antismash.secondarymetabolites.org
- name: GNPS
  role: provides the paired-sample MS/MS spectra job and calculates pairwise modified
    cosine scores for building the MS/MS fingerprint testing matrix
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: k-nearest neighbor classifier
  role: scikit-learn model trained on the labeled BGC fingerprint matrix and used
    to classify MS/MS fingerprints into GCFs (k=3)
  repo: https://github.com/tiagolbiotech/NPOmix_python
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:9caf8c2642fc158a1b5130da7d47e5b40b988f39a1e6aafb6fee2ea6d3dc2cd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - Dereplicator+
  - GNPS
  - MIBiG database
  - MZmine
  - NPOmix
  - antiSMASH
  - k-nearest neighbor classifier
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# bgc-gene-cluster-family-classification

## Summary

Classify a query MS/MS fingerprint into biosynthetic gene cluster families (GCFs) using NPOmix, a k-nearest-neighbor classifier trained on similarity-based BGC fingerprints computed with BiG-SCAPE and GNPS modified cosine scores over paired genome-metabolome samples.

## When to use

Use when you have paired genome-MS/MS samples (e.g. PoDP genomes plus LC-MS/MS spectra), a query MS/MS spectrum (GNPS reference or cryptic), and want to link it to a gene cluster family with precision ~0.929 top-10.

## When NOT to use

- When the query genome or sample has no BGC similar to any BGC in the NPOmix training dataset (e.g. a fungal species with no homologs in the PoDP-derived training set) — the output will contain no answer
- When the input is a single unpaired genome or metabolome with no corresponding paired samples — the classifier requires paired genome-MS/MS co-occurrence to build fingerprints
- When distinguishing structural isomers or regioselectivity of homologous metabolites is required — NPOmix's correlation-based approach lacks this resolution

## Inputs

- Paired genome-MS/MS samples from the Paired Omics Data Platform (PoDP), with genome/metagenome assemblies and LC-MS/MS spectra per sample
- antiSMASH BGC annotations (e.g. antismash_only_gbk.zip from Zenodo record 6637083, 5,421 BGCs in 1,040 genomes)
- BiG-SCAPE similarity network outputs concatenated across classes (hybrids_glocal network)
- GNPS molecular networking job outputs (task=b915d1f78c364bf4be9900047760d95c) with paired sample MS/MS spectra
- Query MS/MS spectrum fingerprint: reference spectrum from GNPS or a cryptic spectrum from a new sample
- GCF label column for training BGCs

## Outputs

- GCF label (gene cluster family assignment) for each query MS/MS fingerprint, ranked by k-nearest neighbors (k=3)
- Top-1 and top-10 precision metrics (reference: 0.818 / 0.929 on the validated PoDP training set)
- Training matrix of BGC fingerprints (rows = maximum similarity scores per genome; columns = genomes)
- Testing matrix of MS/MS fingerprints (rows = query spectra; columns = experimental samples)

## How to apply

Build a training matrix where rows are BGC fingerprints (vector of maximum similarity scores, 1 minus BiG-SCAPE raw distance, of each query BGC to every genome in the paired dataset) and a GCF label column; scores below the similarity cutoff of 0.7 are binarized as absent. Build an analogous testing matrix for each query MS/MS spectrum using GNPS pairwise modified cosine scores against the experimental spectra of each sample (1,040 samples in the reference dataset; Zenodo record 6637083 provides the 5,421 training antiSMASH BGCs from 1,040 genomes/metagenomes). Feed the labeled BGC matrix into a scikit-learn KNN classifier (k = 3 nearest BGC neighbors) and project the MS/MS fingerprints into the same space to obtain GCF labels for each query spectrum; enable similarity plus biosynthetic class features and apply the Jaccard co-occurrence threshold with randomness 0. The rationale is that co-occurrence of metabolite spectra and homologous BGCs across paired samples lets a correlation-based classifier recover links that individual genome-mining tools cannot, achieving top-1 precision 81.8% and top-10 precision 92.9% on the validated training set.

## Related tools

- **NPOmix** (k-nearest-neighbor classifier that assigns gene cluster family labels to query MS/MS fingerprints using similarity BGC and MS/MS fingerprints) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (computes pairwise similarity scores (1 minus raw distance) between BGCs to build the BGC fingerprint training matrix and assign GCFs (with --include_singletons --mibig and Pfam-A.hmm 35.0)) — https://bigscape-corason.secondarymetabolites.org
- **antiSMASH** (produces the training BGC .gbk files that are converted into BGC fingerprints; also used to annotate sample genomes included in the training matrix) — https://antismash.secondarymetabolites.org
- **GNPS** (provides the paired-sample MS/MS spectra job and calculates pairwise modified cosine scores for building the MS/MS fingerprint testing matrix) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **k-nearest neighbor classifier** (scikit-learn model trained on the labeled BGC fingerprint matrix and used to classify MS/MS fingerprints into GCFs (k=3)) — https://github.com/tiagolbiotech/NPOmix_python

## Examples

```
conda activate bigscape; bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- Top-10 precision reaches 0.929 (13 of 14 validated reference MS/MS spectra correctly labeled at top-n=3, k=3) on the PoDP training set
- Top-1 precision reaches 0.818 with randomness=0, similarity + biosynthetic class features, and Jaccard co-occurrence threshold
- BGC fingerprint rows with similarity scores below the 0.7 cutoff are correctly binarized as absent in the training matrix
- The number of columns in both training (BGC) and testing (MS/MS) matrices equals the number of paired samples (e.g. 1,040 genomes/metagenomes in the reference release)
- GCF labels repeat across returned neighbors when multiple BGC nearest neighbors belong to the same gene family — a normal KNN behavior to expect

## Limitations

- NPOmix requires similarity scores; submitting a genome or species with no BGC similar to any training BGC yields no answer
- A correlation-based approach cannot distinguish isomers or regioselectivity of homologous metabolites
- Precision may not be reproducible for every dataset, especially if it differs substantially from the current training set
- Metagenome quality can hinder BGC assembly, leading to fragmented BGCs and fewer outputs than expected
- No RiPP-class validated metabolites are in the current PoDP training data, and paired data for plants is lacking

## Evidence

- [intro] KNN classifier deployed with similarity BGC fingerprints and analogous MS/MS fingerprints to classify gene cluster family candidates: "we deployed a K-Nearest Neighbor (KNN) algorithm that uses similarity BGC fingerprints and analogously similarity MS/MS fingerprints to classify gene cluster family"
- [readme] BGC fingerprint construction: maximum similarity of query BGC to every genome, 1 minus BiG-SCAPE raw distance: "each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison (Fig. 1A) to every other BGC in the same set of genomes to compute similarity scores (1 minus"
- [readme] Similarity cutoff of 0.7 for BGC fingerprint binarization: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [readme] KNN classifier with k=3 nearest BGC neighbors to assign GCF labels to MS/MS fingerprints: "the algorithm can obtain the GCF labels for the k-nearest neighbors to the query MS/MS fingerprint (e.g., for three most similar BGC neighbors, k = 3)"
- [discussion] Precision scores top-1 = 81.8% and top-10 = 92.9% with similarity + biosynthetic class features and Jaccard co-occurrence threshold: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] Training dataset size: 5,421 BGCs in 1,040 genomes/metagenomes: "we have used 5,421 BGCs in 1,040 genomes/metagenomes"
- [readme] GNPS modified cosine scoring for MS/MS fingerprint construction (not full molecular networking): "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
- [discussion] No output when query BGC has no similar training BGC: "by submitting a fungal species that has no BGC similar to any BGC in the NPOmix training dataset (obtained from the PoDP database), the output would contain no answer"
