---
name: knn-bgc-metabolite-linkage-classification
description: Use when you have paired genome/metagenome + LC-MS/MS samples and need
  to link query MS/MS spectra (reference or cryptic) to gene cluster families (GCFs)
  without a GNPS library match.
when_to_use_negative:
- The query organism has no BGC similar to any BGC in the NPOmix PoDP training set
  (e.g., a fungal species) — the output will contain no answer.
- You need to distinguish isomers or regioselectivity of homologs — the correlation-based
  approach lacks that resolution.
- Your dataset is very dissimilar to the PoDP training set (almost exclusively bacteria;
  no RiPP-class validated metabolites, no plant paired data) — precision may not be
  reproducible.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NPOmix
  role: Python/Jupyter implementation of the KNN classifier that trains on BGC fingerprints
    and classifies MS/MS fingerprints into GCF labels
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: antiSMASH
  role: Annotates BGCs in genomes/metagenomes to produce the GenBank files used for
    training and sample BGC inputs
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (1 − raw distance) and assigns GCFs
    using hmmpress-pressed Pfam-A.hmm
- name: GNPS
  role: Computes pairwise modified cosine scores between query and experimental MS/MS
    spectra to build MS/MS fingerprints (not full molecular networking)
- name: scikit-learn
  role: Provides the k-nearest-neighbor classifier trained on the labeled BGC fingerprint
    matrix with k = 3
- name: MIBiG database
  role: Source of expert-annotated validated BGC–metabolite links used to evaluate
    prediction precision
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:18700c4a9932e2e177be4c25a799534bb42f75c1b71024f691cfecfbe896d76e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - MIBiG database
  - NPOmix
  - antiSMASH
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# knn-bgc-metabolite-linkage-classification

## Summary

Train and apply an NPOmix k-nearest-neighbor classifier that connects MS/MS fragmentation spectra to biosynthetic gene clusters (BGCs) by comparing similarity-based BGC fingerprints against similarity-based MS/MS fingerprints from paired genome–metabolome samples.

## When to use

Use when you have paired genome/metagenome + LC-MS/MS samples and need to link query MS/MS spectra (reference or cryptic) to gene cluster families (GCFs) without a GNPS library match.

## When NOT to use

- The query organism has no BGC similar to any BGC in the NPOmix PoDP training set (e.g., a fungal species) — the output will contain no answer.
- You need to distinguish isomers or regioselectivity of homologs — the correlation-based approach lacks that resolution.
- Your dataset is very dissimilar to the PoDP training set (almost exclusively bacteria; no RiPP-class validated metabolites, no plant paired data) — precision may not be reproducible.

## Inputs

- antiSMASH BGC GenBank files (training set from antismash_only_gbk.zip at Zenodo record 6637083, plus sample BGCs in antismash_gbk_only folder)
- BiG-SCAPE pairwise similarity network outputs per biosynthetic class (e.g., *_hybrids_glocal files)
- GNPS job output of merged experimental MS/MS spectra for ~1,040 paired genome–metabolome samples from the Paired Omics Data Platform (PoDP)
- Query MS/MS spectra (GNPS reference spectra or cryptic spectra, e.g., MGF format)
- hmmpress-pressed Pfam-A.hmm database (Pfam 35.0)

## Outputs

- BGC fingerprint training matrix (5,421 BGCs × 1,040 genome/metagenome columns plus GCF label column)
- MS/MS fingerprint testing matrix (query spectra × 1,040 sample columns)
- Predicted metabolite–BGC/GCF links (e.g., npomix_predictions.csv, siderophore_npomix_links.tsv)
- Precision results at top-1 (81.8%) and top-10 (92.9%) against MIBiG/PoDP validated links (e.g., npomix_precision_results.csv)

## How to apply

Annotate BGCs with antiSMASH, run BiG-SCAPE with hmmpress-pressed Pfam-A.hmm (Pfam 35.0), --include_singletons and --mibig, and concatenate per-class network outputs (e.g., *_hybrids_glocal) with concat_bigscape.sh. Build the BGC fingerprint training matrix by computing, for each query BGC, the maximum BiG-SCAPE similarity (1 − raw distance) against each of ~1,040 genomes/metagenomes, treating scores below the 0.7 similarity cutoff as absent; append a GCF label column and train a scikit-learn KNN classifier (k = 3). Build the MS/MS fingerprint testing matrix by computing pairwise GNPS modified cosine scores between each query spectrum (e.g., 15 reference spectra) and the merged experimental spectra of each sample — deliberately not using full GNPS molecular networking. Project each MS/MS fingerprint into the trained KNN space, take the GCF labels of the 3 nearest BGC neighbors (repeated labels allowed), and optionally retain links passing the Jaccard co-occurrence threshold of 0.7; optionally append biosynthetic class or manually annotated substructure features, which the article shows improve performance.

## Related tools

- **NPOmix** (Python/Jupyter implementation of the KNN classifier that trains on BGC fingerprints and classifies MS/MS fingerprints into GCF labels) — https://github.com/tiagolbiotech/NPOmix_python
- **antiSMASH** (Annotates BGCs in genomes/metagenomes to produce the GenBank files used for training and sample BGC inputs)
- **BiG-SCAPE** (Computes pairwise BGC similarity scores (1 − raw distance) and assigns GCFs using hmmpress-pressed Pfam-A.hmm)
- **GNPS** (Computes pairwise modified cosine scores between query and experimental MS/MS spectra to build MS/MS fingerprints (not full molecular networking))
- **scikit-learn** (Provides the k-nearest-neighbor classifier trained on the labeled BGC fingerprint matrix with k = 3)
- **MIBiG database** (Source of expert-annotated validated BGC–metabolite links used to evaluate prediction precision)

## Examples

```
conda activate bigscape && bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- Against validated MIBiG/PoDP links with the Jaccard co-occurrence threshold of 0.7 and k = 3 (similarity + biosynthetic class features), expect top-1 precision ≈ 81.8% and top-10 precision ≈ 92.9% (13 of 14 reference spectra correctly labeled at top-3).
- Fingerprint matrices must have 1,040 columns (one per paired sample); BGC matrix rows ≈ 5,421, testing matrix rows = number of query spectra (e.g., 15 reference spectra).
- Similarity scores below the 0.7 cutoff must be treated as absent (0.0) in BGC fingerprints; identical BGCs score 1.0.
- In the siderophore workflow, the m/z 780 MassQL-filtered spectrum should link with 100% Jaccard similarity to the GCF containing the Achromobacter xylosoxidans NH44784-1996 siderophore BGC.
- Repeated GCF labels among the k = 3 returned neighbors are expected and valid KNN behavior, not an error.

## Limitations

- NPOmix cannot distinguish isomers or regioselectivity of homologs.
- Precision score may not be reproducible for datasets dissimilar to the current PoDP training set.
- Poor-quality metagenomes hinder BGC assembly, producing fewer, fragmented BGCs.
- No RiPP-class validated metabolites and almost exclusively bacterial data in the training set; plant paired data are lacking.
- Submitting an organism with no BGC similar to any training-set BGC yields no answer.
- Putative new links (e.g., orfamides) require proper structure elucidation; predictions were not confirmed by co-elution with standards.

## Evidence

- [abstract] knn abstract quote: "Here, we present a k-nearest neighbor classifier to systematically connect mass spectrometry fragmentation spectra to their corresponding biosynthetic gene clusters"
- [discussion] precision metrics: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] fingerprint construction readme: "we identify the maximum similarity of the query BGC to one of the many BGCs in each genome (which can be considered a pool of BGCs) in the dataset"
- [readme] cutoff readme: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [readme] gnps scope readme: "we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score between a pair of MS/MS spectra"
- [readme] knn k readme: "so the algorithm can obtain the GCF labels for the k-nearest neighbors to the query MS/MS fingerprint (e.g., for three most similar BGC neighbors, k = 3)"
- [discussion] isomer limitation: "NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs"
