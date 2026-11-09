---
name: top-percentile-pair-selection
description: Use when a full NxN spectral similarity matrix (e.g. 12,797 x 12,797)
  has been computed and the goal is to quantify how well each score's top matches
  reflect molecular structural similarity.
when_to_use_negative:
- You want correlation across the full score range rather than the extreme high-scoring
  end — the paper's Fig 3 analyses score distributions separately from this top-tail
  comparison
- Spectra lack InChIKey annotations, so Tanimoto structural similarity cannot be derived
  for the paired molecules
- The similarity matrix is sparse or pre-thresholded, so the full ~82M-pair population
  needed to define a genuine top 0.1% is unavailable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: computes cosine and modified cosine similarity matrices and handles peak/loss
    filtering before top-pair selection
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: provides the Spec2Vec similarity score whose matrix's top 0.1% pairs are compared
    against the cosine-based scores
  repo: https://github.com/iomega/spec2vec
- name: Word2Vec
  role: underlying embedding model (via gensim) from which Spec2Vec spectrum vectors
    are derived for the Spec2Vec similarity matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:c3e1890a1f335493646c7111f3ae68cf51db05a0f28f2263bacad7a7f8786a31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Word2Vec
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# top-percentile-pair-selection

## Summary

Selects the very highest-scoring spectral pairs (top 0.1% of all ~82M non-self pairs in the UniqueInchikey dataset) from a full pairwise similarity matrix, so that the structural (Tanimoto) similarity of those best-matching pairs can be averaged per score and compared across Spec2Vec, cosine, and modified cosine as in Fig 3B of the paper.

## When to use

Use when a full NxN spectral similarity matrix (e.g. 12,797 x 12,797) has been computed and the goal is to quantify how well each score's top matches reflect molecular structural similarity.

## When NOT to use

- You want correlation across the full score range rather than the extreme high-scoring end — the paper's Fig 3 analyses score distributions separately from this top-tail comparison
- Spectra lack InChIKey annotations, so Tanimoto structural similarity cannot be derived for the paired molecules
- The similarity matrix is sparse or pre-thresholded, so the full ~82M-pair population needed to define a genuine top 0.1% is unavailable

## Inputs

- 12,797 x 12,797 similarity matrix from spec2vec.Spec2Vec (Word2Vec model trained on UniqueInchikey, Zenodo record 3978054)
- 12,797 x 12,797 cosine and modified cosine similarity matrices from matchms
- UniqueInchikey dataset (12,797 spectra with InChIKey annotations, Zenodo record 3979074)
- InChIKey-based molecular structures per spectrum for Tanimoto computation

## Outputs

- Top 0.1% highest-scoring spectrum-pair sets per score (~80,000 pairs per score)
- avg_tanimoto_scores.tsv with one average Tanimoto structural similarity value per score, in Fig 3B layout

## How to apply

Compute the full pairwise similarity matrices for each score (Spec2Vec via spec2vec.Spec2Vec on spectra converted to peak/loss-word documents; cosine and modified cosine via matchms, ignoring peaks with relative intensity < 0.01 and matching neutral losses between 5.0 and 200.0 Da for modified cosine). Flatten each matrix while excluding self-matches, then extract the top 0.1% of all ~82M spectrum pairs as the highest-scoring candidate pairs. For each per-score pair set, compute the average Tanimoto structural similarity derived from the paired molecules' InChIKey-based structures, and write the three per-score averages to avg_tanimoto_scores.tsv in the per-score layout of Fig 3B. Rationale: comparing scores only at the extreme high end tests whether a high spectral similarity score reflects a high structural similarity between molecules, the paper's core question.

## Related tools

- **matchms** (computes cosine and modified cosine similarity matrices and handles peak/loss filtering before top-pair selection) — https://github.com/matchms/matchms
- **spec2vec** (provides the Spec2Vec similarity score whose matrix's top 0.1% pairs are compared against the cosine-based scores) — https://github.com/iomega/spec2vec
- **Word2Vec** (underlying embedding model (via gensim) from which Spec2Vec spectrum vectors are derived for the Spec2Vec similarity matrix)

## Examples

```
scores = calculate_scores(spectra, spectra, Spec2Vec(model=model, intensity_weighting_power=0.5, allow_missing=False))
```

## Evaluation signals

- The Spec2Vec column of avg_tanimoto_scores.tsv must show a higher average Tanimoto similarity than both the cosine and modified cosine columns, reproducing the paper's finding
- Each extracted top-pair set must contain ~80,000 pairs (0.1% of ~82M pairs) with self-matches excluded
- The TSV must follow the per-score layout of Fig 3B with exactly three score columns (Spec2Vec, cosine, modified cosine)
- All extracted pairs must have valid InChIKey-derived structures so Tanimoto similarity is defined for every pair

## Limitations

- Cosine and modified cosine require precursor-m/z pre-selection of candidate matches before scoring, whereas Spec2Vec scores all pairs; the top-tail sets are thus derived under slightly different scoring mechanics
- Spec2Vec needs training data: it requires that the training set includes a large fraction of the fragment ions and losses present in the query spectra
- Tanimoto on InChIKey-derived structures is a practical proxy; the best measure of structural similarity lies outside the study's scope

## Evidence

- [abstract] we show how Spec2Vec scores correlate better with structural similarity than cosine-based scores: "we show how Spec2Vec scores correlate better with structural similarity than cosine-based scores"
- [intro] In order to quantify how well different spectral similarity scores correlate with structural similarity: "In order to quantify how well different spectral similarity scores correlate with structural similarity"
- [discussion] For both the cosine and modified cosine score calculations we ignored all peaks with relative intensities <0.01 compared to the highest intensity peak.: "For both the cosine and modified cosine score calculations we ignored all peaks with relative intensities <0.01 compared to the highest intensity peak."
- [discussion] In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”.: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [readme] One of the strengths of matchms is its extensibility, allowing users to integrate custom similarity measures.: [evidence span withheld — anchor status: external_doc]
