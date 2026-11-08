---
name: intensity-weighted-spectrum-vector-scoring
description: Use when comparing MS/MS query spectra against a library (e.g. cleaned
  AllPositive GNPS dataset) and peak-based cosine scores underperform, or when you
  need similarity scores that track structural similarity across instruments and conditions.
when_to_use_negative:
- No training data are available for the target domain — cosine scores need no model,
  but Spec2Vec 'needs training data to learn the fragment peak relationships'
- The training set lacks a large fraction of the fragment ions and losses seen in
  the query data (small datasets or finer m/z decimals aggravate the missing-words
  problem)
- The data are GC-MS spectra — performance has only been demonstrated on LC-MS data
tools:
- name: Spec2Vec
  role: Provides document conversion (SpectrumDocument), calc_vector for intensity-weighted
    spectrum vectors, and the cosine-of-vectors similarity score
  repo: github.com/iomega/spec2vec
- name: gensim Word2Vec
  role: Trains the word2vec model on all spectrum documents from which peak/loss word
    embeddings are taken
- name: matchms
  role: Imports, cleans, and filters spectra (m/z range [0,1000], >=10 peaks, neutral-loss
    words) before scoring
  repo: github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:253317fba346409e789e7e95a973e4e3a21d840c4c204d8214a0b1918ad52cdd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Spec2Vec
  - gensim Word2Vec
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# intensity-weighted-spectrum-vector-scoring

## Summary

Computes Spec2Vec spectral similarity by converting MS/MS spectra into documents of peak/loss words, then scoring pairs via the cosine between intensity-weighted sums of Word2Vec word vectors. It enables library matching and structural-analogue detection that correlates better with structural (Tanimoto) similarity than cosine peak matching.

## When to use

Use when comparing MS/MS query spectra against a library (e.g. cleaned AllPositive GNPS dataset) and peak-based cosine scores underperform, or when you need similarity scores that track structural similarity across instruments and conditions.

## When NOT to use

- No training data are available for the target domain — cosine scores need no model, but Spec2Vec 'needs training data to learn the fragment peak relationships'
- The training set lacks a large fraction of the fragment ions and losses seen in the query data (small datasets or finer m/z decimals aggravate the missing-words problem)
- The data are GC-MS spectra — performance has only been demonstrated on LC-MS data

## Inputs

- Processed positive-ionization MS/MS library spectra (e.g. cleaned AllPositive GNPS dataset from Zenodo record 3979010, ~94,320 spectra)
- Query MS/MS spectra (e.g. 1000 queries with precursor m/z metadata for candidate pre-selection)
- A trained gensim Word2Vec model (e.g. Zenodo record 4173596, trained on AllPositive dataset)

## Outputs

- Spectrum vectors (intensity-weighted sums of word vectors) for library and query spectra
- Cosine similarity scores between query–candidate spectrum-vector pairs
- Threshold sweep table (library_matching_results.csv) with accuracy and true/false-positive match rates
- Accuracy and TP/FP-rate curves versus similarity threshold (reproducing Fig 4)

## How to apply

First, with matchms, keep peaks with m/z in [0, 1000], discard spectra with fewer than 10 peaks, and add neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx' words; for Spec2Vec the maximum number of kept peaks per spectrum is capped at 0.5·parent_mass. Convert each spectrum into a document where every peak is a word encoding its m/z position to a defined decimal precision, then train a gensim Word2Vec model on all library documents (94,320 library spectra in the AllPositive run). Compute each spectrum's vector as the intensity-weighted sum of its word vectors using Spec2Vec's calc_vector, and score query–candidate pairs as the cosine between the two spectrum vectors. For library matching, pre-select candidates by precursor-m/z within 1 ppm tolerance, then pick the highest-scoring candidate above each similarity threshold; sweep the threshold to build accuracy and true/false-positive rate curves. Optionally apply a missing-fraction threshold (<0.05) so spectra far outside the learned peak/loss vocabulary do not receive misleading Spec2Vec scores.

## Related tools

- **Spec2Vec** (Provides document conversion (SpectrumDocument), calc_vector for intensity-weighted spectrum vectors, and the cosine-of-vectors similarity score) — github.com/iomega/spec2vec
- **gensim Word2Vec** (Trains the word2vec model on all spectrum documents from which peak/loss word embeddings are taken)
- **matchms** (Imports, cleans, and filters spectra (m/z range [0,1000], >=10 peaks, neutral-loss words) before scoring) — github.com/matchms/matchms

## Examples

```
from spec2vec import calc_vector; v_query = calc_vector(word2vec_model, query_spectrum_document, intensity_weighting_power=1.0)
```

## Evaluation signals

- On the AllPositive split, best threshold must yield up to 88% of the 1000 query spectra correctly matched to library entries
- Accuracy and TP/FP-rate curves versus threshold must qualitatively reproduce the published Fig 4 shapes
- Spec2Vec must show a better true/false-positive ratio than cosine-based matching at all thresholds
- Spectrum vectors must be intensity-weighted sums over word vectors, with kept peaks capped at 0.5·parent_mass
- Missing-fraction check (<0.05) should be applied so out-of-model spectra do not receive scores from the learned vocabulary

## Limitations

- Spec2Vec requires a trained model, unlike cosine scores, and the training set must include a large fraction of the query fragment/loss features
- Missing words become a real issue when models are trained on smaller datasets or when more m/z decimals are included in the peak words
- Network analysis applications may require a much larger training set or retraining including new experimental spectra
- Scores for spectra far outside the learned peaks/losses can be misleading unless a missing-fraction threshold (e.g. <0.05) is set

## Evidence

- [discussion] spectrum vector as weighted sum: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [discussion] cosine between spectrum vectors: "For the similarity between two spectra we then compute the cosine score between two spectrum vectors: g(s1,s2)=cosine(vS1,vS2)"
- [discussion] peak cap scaling with parent mass: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] loss words added: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [intro] candidate preselection by precursor m/z: "potentially matching spectra were pre-selected based on precursor-m/z matches (tolerance = 1ppm) before the highest scoring candidate above a similarity threshold was chosen."
- [readme] score from learned embeddings: "The spectral similarity score is based on spectral embeddings learnt from the fragmental relationships within a large set of spectral data."
