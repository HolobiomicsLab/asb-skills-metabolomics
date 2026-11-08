---
name: tanimoto-structural-similarity-computation
description: Use when spectral similarity scores between MS/MS spectrum pairs must
  be validated against ground-truth structural similarity, e.g. averaging Tanimoto
  similarity over top-0.1% scoring pairs or over molecular-network clusters.
when_to_use_negative:
- Spectra lack InChIKey annotations or cannot be linked to a structure via PubChem
  lookup — no structural ground truth exists
- You need the theoretically best structural-similarity measure rather than a practical
  standard; the authors explicitly place finding the best measure outside the study's
  scope and adopt Tanimoto as a convergence point
- Assessing whether two spectra are similar without any molecular structure available
  (Tanimoto needs derived fingerprints, not raw peaks)
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: matchms
  role: Imports, cleans and compares MS/MS spectra; supports fingerprint-based similarity
    comparisons alongside cosine-related scores, providing the spectral scores whose
    top pairs are then validated by Tanimoto
  repo: https://github.com/matchms/matchms
- name: Spec2Vec
  role: Produces the learned-embedding spectral similarity scores that are benchmarked
    against Tanimoto structural similarity
  repo: https://github.com/iomega/spec2vec
- name: spec2vec_gnps_data_analysis
  role: Analysis notebooks that compute and plot Tanimoto-based cluster quality and
    score-correlation curves for the GNPS dataset
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
merged_aliases:
- tanimoto-similarity-structural-matching
merged_alias_records:
- alias: tanimoto-structural-similarity-quantification
  slug: tanimoto-structural-similarity-quantification
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:3ea9ae6b893316c2b85e29b074bf28678d1be2a32082646afc20fee311b0782e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - ChemmineR
  - LOTUS
  - R
  - Spec2Vec
  - fmcsR
  - matchms
  - spec2vec_gnps_data_analysis
  - uafR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tanimoto-structural-similarity-computation

## Summary

Computes molecular structural similarity as Tanimoto similarity derived from InChIKey-based structures, used in the Spec2Vec paper to quantify how well spectral similarity scores (Spec2Vec, cosine, modified cosine) correlate with structural relatedness of fragmented molecules.

## When to use

Use when spectral similarity scores between MS/MS spectrum pairs must be validated against ground-truth structural similarity, e.g. averaging Tanimoto similarity over top-0.1% scoring pairs or over molecular-network clusters.

## When NOT to use

- Spectra lack InChIKey annotations or cannot be linked to a structure via PubChem lookup — no structural ground truth exists
- You need the theoretically best structural-similarity measure rather than a practical standard; the authors explicitly place finding the best measure outside the study's scope and adopt Tanimoto as a convergence point
- Assessing whether two spectra are similar without any molecular structure available (Tanimoto needs derived fingerprints, not raw peaks)

## Inputs

- MS/MS spectra with InChIKey annotations (e.g. UniqueInchikey dataset, 12,797 spectra from Zenodo 3979074; AllPositive dataset, 95,320 spectra from Zenodo 3979010)
- All-pairs spectral similarity scores (Spec2Vec, cosine, modified cosine) for the same spectra
- Structures or fingerprints derived from InChIKeys (obtainable via PubChem lookup with pubchempy)

## Outputs

- Per-pair or per-cluster Tanimoto structural similarity values
- Average Tanimoto per spectral score for top-0.1% pairs (e.g. avg_tanimoto_scores.tsv in Fig 3B layout)
- Well-clustered vs poorly-clustered spectrum fractions (mean Tanimoto >= 0.5 vs < 0.5) per score

## How to apply

Convert the InChIKey annotations attached to each spectrum into molecular structures (the paper linked spectra to InChIKeys via an automated PubChem lookup with pubchempy, succeeding for 128,042 of 154,820 GNPS spectra), then compute pairwise Tanimoto similarity of the corresponding structural fingerprints. For score-validation, restrict to the comparison set at hand — e.g. the ~82 million non-self pairs of the 12,797-spectrum UniqueInchikey dataset — extract the top 0.1% of pairs per score, and average the Tanimoto values per score to compare Spec2Vec against cosine and modified cosine (which ignore peaks with relative intensity <0.01, with neutral losses 5.0–200.0 Da). For molecular networking (top-10 links per node, Louvain split of large poorly connected clusters), compute the mean Tanimoto across each cluster's edges and classify a spectrum as well-clustered when mean Tanimoto >= 0.5. The rationale is that Tanimoto is the practical measure the field converges on for structural similarity, even though the 'best' measure was declared out of scope by the authors.

## Related tools

- **matchms** (Imports, cleans and compares MS/MS spectra; supports fingerprint-based similarity comparisons alongside cosine-related scores, providing the spectral scores whose top pairs are then validated by Tanimoto) — https://github.com/matchms/matchms
- **Spec2Vec** (Produces the learned-embedding spectral similarity scores that are benchmarked against Tanimoto structural similarity) — https://github.com/iomega/spec2vec
- **spec2vec_gnps_data_analysis** (Analysis notebooks that compute and plot Tanimoto-based cluster quality and score-correlation curves for the GNPS dataset) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Evaluation signals

- For the top 0.1% of spectrum pairs (~80,000 pairs from ~82M) in the UniqueInchikey dataset, the Spec2Vec column must show higher average Tanimoto than cosine and modified cosine columns, matching Fig 3B's finding
- Every scored spectrum pair must carry InChIKey-derived structures; verify spectra without InChIKeys were excluded rather than silently assigned Tanimoto 0
- Self-matches must be excluded from the ~82M pair set before taking the top 0.1%
- In molecular networking, clusters should be split with Louvain and per-cluster mean Tanimoto compared against the 0.5 well/poorly-clustered threshold
- InChIKey linkage coverage should be checked (the paper's PubChem lookup succeeded for 128,042/154,820 spectra), since unresolved structures reduce the evaluable pair set

## Limitations

- Tanimoto from InChIKey-derived structures is a practical proxy; the authors state the best structural-similarity measure lies outside the scope of the study
- Requires complete InChIKey coverage; spectra without annotations (or without a PubChem hit) cannot be evaluated and must be filtered out
- Tanimoto averages over a score's top pairs are sensitive to the chosen top fraction (0.1%) and, in clustering, to the top-10 links-per-node and mean-Tanimoto >= 0.5 thresholds

## Evidence

- [discussion] Finding the best measure to define structural similarity between two molecules lies outside the scope of this study; most recent studies converge to the Tanimoto similarity: "Finding the best measure to define structural similarity between two molecules lies outside the scope of this study; most recent studies converge to the Tanimoto similarity"
- [discussion] an extensive automated lookup search was run against PubChem (via pubchempy): "an extensive automated lookup search was run against PubChem [35] (via pubchempy [36]). As a result, 128,042 out of 154,820 spectra could be linked to an InChIKey"
- [intro] In order to quantify how well different spectral similarity scores correlate with structural similarity: "In order to quantify how well different spectral similarity scores correlate with structural similarity"
- [readme] This encompasses not only common Cosine-related scores but also molecular fingerprint-based comparisons and other metadata-related assessments.: [evidence span withheld — anchor status: external_doc]
